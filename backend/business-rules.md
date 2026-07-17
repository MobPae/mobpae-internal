# Business Rules

All core business rules are enforced in the backend services. This document collects them in one place.

---

## 1. Eligibility Rules (EligibilityService)

The following checks gate whether an employee can submit an advance request:

| Rule | Detail |
|---|---|
| KYC required | All KYC documents must be in `VERIFIED` status |
| Bank account required | Bank account must be verified |
| Minimum salary | Salary must be ≥ `minimumSalaryInHand` from product config |
| Minimum tenure | Employee's `joiningDate` must be ≥ `minimumTenureMonths` months ago |
| No active advance | Employee cannot have another advance in-flight (configurable via `maxRequestsPerCycle`) |
| Cooldown period | After a repayment, employee must wait `cooldownDays` before a new request |
| Amount limits | Requested amount ≥ `minimumAdvanceAmount` and ≤ computed advance limit |
| Employee active | `employmentStatus` must be `ACTIVE` |
| Employer approved | Employer status must be `APPROVED` |

---

## 2. Advance Limit Computation

The employee's maximum advance amount is computed dynamically from multiple sources:

```
platformAdvancePercentage   = LoanProductConfig.eligibilityRules.maximumAdvancePercentage
platformMaxAdvanceAmount    = LoanProductConfig.eligibilityRules.maximumAdvanceAmount (₹ cap)
employerPercentageOverride  = EmployerProductConfig.maximumAdvancePercentageOverride
employerAmountOverride      = EmployerProductConfig.maximumAdvanceAmountOverride
hardCeilingPercentage       = LoanProductConfig.eligibilityRules.hardCeilingPercentage
adminLoanLimit              = LoanLimit.maximumEligibleAmount (set by admin per employee)

Step 1: Effective percentage = employerPercentageOverride ?? platformAdvancePercentage
Step 2: Cap to hardCeilingPercentage (employer cannot exceed this)
Step 3: Percentage-based limit = salary × effectivePercentage / 100
Step 4: Cap to platformMaxAdvanceAmount (platform-wide ₹ ceiling)
Step 5: Apply employerAmountOverride if set (admin-granted special case)
Step 6: Apply adminLoanLimit if set (per-employee admin cap)
Step 7: Final limit = min of all applicable ceilings
```

---

## 3. Interest Calculation

**Formula (Simple Daily Interest):**

```
interestFreeThreshold    = min(salary × platformAdvancePct%, platformMaxAmt)
interestFreeAmount       = min(principal, interestFreeThreshold)
interestBearingAmount    = principal - interestFreeAmount

interestAmount           = interestBearingAmount × (annualRate / 100) × (interestDays / 365)
processingFee            = principal × processingFeeRate
gstAmount                = (interestAmount + processingFee) × gstRate

totalRepayment           = principal + interestAmount + processingFee + gstAmount
```

**Current defaults (from seed/product config):**
- Annual interest rate: **36%**
- Processing fee rate: **0%**
- GST rate: **0%**
- Interest-free threshold: based on platform advance % × salary

**Interest days** are computed at submission time as the number of days from submission to the next payroll recovery date, using the employer's payroll calendar.

---

## 4. Payroll Recovery Date

```
Rule:
  If submissionDay < payrollCutoffDate:
    recover on THIS month's payrollDate
  Else:
    recover on NEXT month's payrollDate

Example:
  payrollDate = 28, payrollCutoffDate = 21
  Submit on 15th July → recover 28th July (15 < 21)
  Submit on 25th July → recover 28th August (25 >= 21)
```

Implemented in `PricingService.resolveRecoveryDate()`.

---

## 5. Platform Fee Rules

| Rule | Detail |
|---|---|
| Triggered on | Employer approval of a loan application |
| Amount | ₹175 (configurable in `Settings` table via key `PLATFORM_FEE_CONFIG`) |
| Paid by | Employee |
| Payment method | Razorpay order (online payment) |
| Required before | Admin review / disbursal |
| Can be waived | Yes — by admin only (records `waivedAt`, `waivedBy`) |
| Status flow | PENDING_PAYMENT → PAID / FAILED / EXPIRED / WAIVED |
| One fee per application | Unique constraint on `(loanApplicationId, feeType)` |

---

## 6. Employer Amount Override Rule

Employers can configure their advance percentage (within MobPae's hard ceiling). If an employer approves an amount higher than what the standard product config would allow:

- The employer bears the 36% annual interest on the **extra amount** (amount above standard limit)
- This is a commercial agreement between MobPae and the employer
- Currently tracked in `EmployerProductConfig.maximumAdvancePercentageOverride`
- Interest billing for override amounts is handled via the settlement process (not automated yet)

---

## 7. Loan Application Status Transitions

| From | To | Who | Trigger |
|---|---|---|---|
| — | `SUBMITTED` | Employee | Submit application |
| `SUBMITTED` | `EMPLOYER_APPROVED` | Employer | Employer approves |
| `SUBMITTED` | `EMPLOYER_REJECTED` | Employer | Employer rejects |
| `SUBMITTED` | `CANCELLED` | Employee | Employee cancels |
| `EMPLOYER_APPROVED` | `AWAITING_PLATFORM_FEE_PAYMENT` | System | Fee record created |
| `AWAITING_PLATFORM_FEE_PAYMENT` | `READY_FOR_DISBURSAL` | System | Fee paid or waived |
| `READY_FOR_DISBURSAL` | `ADMIN_REJECTED` | Admin | Admin rejects |
| `READY_FOR_DISBURSAL` | `DISBURSED` | System | Disbursal succeeds |
| `DISBURSED` | `REPAYMENT_SCHEDULED` | System | Repayment record created |
| `REPAYMENT_SCHEDULED` | `REPAID` | Admin | Admin marks repayment paid |
| Any non-terminal | `EXPIRED` | System (scheduled job) | Timeout |

---

## 8. Disbursal Rules

| Rule | Detail |
|---|---|
| Who initiates | Admin only |
| Bank details | Frozen on Disbursal record at creation; even if employee changes bank later, old disbursal is unaffected |
| Retry | `retryCount` tracks how many times disbursal was attempted |
| Provider | Currently manual / NEFT; Razorpay Payout integration planned |

---

## 9. Settlement Rules

| Rule | Detail |
|---|---|
| Frequency | One settlement per employer per payroll cycle |
| Unique constraint | `(employerId, cycleDate)` — prevents duplicate settlements |
| Amounts | Frozen at generation time — never recalculated |
| Partial payment | Supported via `SettlementPayment` records; outstanding amount decremented |
| Line items | One `SettlementLineItem` per repayment; each repayment can only be in one settlement |
| Employee data | Employee name and application number frozen on line item (even if employee data changes later) |

---

## 10. KYC and Bank Account Rules

| Rule | Detail |
|---|---|
| Documents | Aadhaar, PAN, Salary Slip (one per type per employee) |
| Resubmission | Employee can re-upload; new upload replaces the rejected document |
| Bank account | One per employee (`@unique` on `employeeId`) |
| File storage | All files stored in R2; DB stores only the file key (not URL) |
| Access | Files accessed only via 15-minute signed URLs generated by backend |

---

## 11. One Active Advance Per Employee

By default, an employee can have only **one** active advance at a time. "Active" means any status that is not a terminal state (`REPAID`, `ADMIN_REJECTED`, `EMPLOYER_REJECTED`, `CANCELLED`, `EXPIRED`).

This is checked during eligibility computation using `maxRequestsPerCycle` from the product config (default: 1).

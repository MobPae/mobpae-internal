# Database Enums

All enum values are stored as strings in PostgreSQL. Defined in `prisma/schema.prisma`.

---

## Role

Used on `User.role`. Determines which portal the user accesses and what endpoints they can call.

| Value | Who | Portal |
|---|---|---|
| `ADMIN` | MobPae operations team | mobpae-admin |
| `EMPLOYER` | HR/finance manager | mobpae-employer |
| `EMPLOYEE` | Salaried employee | mobpae-app |

---

## EmployerStatus

Lifecycle of an employer account.

| Value | Meaning |
|---|---|
| `PENDING` | Submitted onboarding form; not yet reviewed |
| `APPROVED` | MobPae approved; employer can log in and manage employees |
| `REJECTED` | Onboarding denied |
| `ACTIVE` | Employer is live and employees can use the platform |
| `SUSPENDED` | Temporarily blocked (e.g., payment issues); employees cannot apply |

---

## EmployerRiskStatus

Creditworthiness signal based on settlement history.

| Value | Meaning |
|---|---|
| `GOOD` | No overdue settlements |
| `WARNING` | Settlement overdue but within grace period |
| `BLOCKED` | Seriously overdue; new advances may be paused for this employer's employees |

Updated by admin via `POST /employer-settlements/check-risk/:employerId`.

---

## EmployeeStatus

Tracks employment state.

| Value | Meaning |
|---|---|
| `ACTIVE` | Employed; can use the platform |
| `INACTIVE` | Resigned / terminated; cannot apply for advances |

---

## SelfieStatus

Status of the employee's selfie photo used for identity verification.

| Value | Meaning |
|---|---|
| `PENDING` | Selfie uploaded; awaiting admin review |
| `VERIFIED` | Admin confirmed identity |
| `REJECTED` | Mismatch or poor quality; employee must resubmit |

---

## KycStatus

Status of a single KYC document (each doc has its own status).

| Value | Meaning |
|---|---|
| `PENDING` | Uploaded; not yet reviewed |
| `VERIFIED` | Admin approved |
| `REJECTED` | Admin rejected; `rejectionNote` explains why |

---

## LoanProductType

Product catalog entries.

| Value | Display Name | Status |
|---|---|---|
| `SA` | Salary Advance | ✅ Live |
| `PL` | Personal Loan | 🔜 Coming soon |
| `HL` | Home Loan | 🔜 Coming soon |
| `VL` | Vehicle Loan | 🔜 Coming soon |
| `EL` | Education Loan | 🔜 Coming soon |
| `CL` | Credit Line | 🔜 Coming soon |

---

## LoanApplicationStatus

The main status machine for a loan application. Follows a strict forward-only progression.

| Value | What it means |
|---|---|
| `SUBMITTED` | Employee submitted; awaiting employer review |
| `EMPLOYER_APPROVED` | Employer approved; platform fee payment required |
| `EMPLOYER_REJECTED` | Employer rejected the request |
| `AWAITING_PLATFORM_FEE_PAYMENT` | Fee order created; waiting for Razorpay confirmation |
| `READY_FOR_DISBURSAL` | Fee paid; awaiting admin disbursal |
| `ADMIN_REJECTED` | Admin rejected at any point before disbursal |
| `DISBURSED` | Admin created the disbursal record |
| `REPAYMENT_SCHEDULED` | Repayment record created; employee owes on dueDate |
| `REPAID` | Employer settled; repayment marked PAID |

---

## PurposeCategory

Why the employee needs the advance. Selected at application time.

| Value |
|---|
| `MEDICAL` |
| `EDUCATION` |
| `HOUSE_RENT` |
| `UTILITY_BILLS` |
| `TRAVEL` |
| `EMERGENCY` |
| `OTHER` |

---

## DisbursalStatus

Tracks the payout process.

| Value | Meaning |
|---|---|
| `PENDING` | Disbursal record created; not yet sent |
| `PROCESSING` | Transfer initiated |
| `SUCCESS` | Money received by employee |
| `FAILED` | Transfer failed; admin can retry |
| `CANCELLED` | Admin cancelled the disbursal |

---

## RepaymentStatus

| Value | Meaning |
|---|---|
| `SCHEDULED` | Created; due on `dueDate` |
| `PAID` | Employer paid and settlement marked complete |
| `OVERDUE` | Past `dueDate` with no settlement payment |

---

## EmployerSettlementStatus

| Value | Meaning |
|---|---|
| `DRAFT` | Auto-created; not yet finalized |
| `GENERATED` | Sent to employer; payment expected |
| `PARTIALLY_PAID` | One or more payments received but total not yet met |
| `PAID` | Fully paid; all line-item repayments marked PAID |
| `OVERDUE` | Due date passed with outstanding amount remaining |
| `CANCELLED` | Voided by admin |

---

## LoanApplicationFeeStatus

Status of the ₹175 platform fee for a specific application.

| Value | Meaning |
|---|---|
| `PENDING_PAYMENT` | Fee order created in Razorpay; awaiting payment |
| `PAID` | Razorpay webhook confirmed payment |
| `FAILED` | Payment failed |
| `EXPIRED` | Order expired without payment (Razorpay default: 15 min) |
| `REFUNDED` | Payment refunded |
| `WAIVED` | Admin waived the fee |

---

## PaymentProvider (on Disbursal)

How money is sent to the employee.

| Value | Meaning |
|---|---|
| `RAZORPAY_PAYOUT` | Via Razorpay Payouts API |
| `CASHFREE` | Via Cashfree Payouts |
| `BANK_TRANSFER` | Manual NEFT/RTGS |
| `NACH` | Auto-debit mandate |
| `INTERNAL` | Internal transfer (for testing) |

Currently all disbursals are done as `INTERNAL` (manual bank transfer tracked outside system).

---

## SettlementPaymentMethod (on SettlementPayment)

How employer pays the settlement.

| Value |
|---|
| `NEFT` |
| `RTGS` |
| `IMPS` |
| `UPI` |
| `NACH` |
| `CHEQUE` |

---

## SettlementPaymentStatus (on SettlementPayment)

| Value | Meaning |
|---|---|
| `PENDING` | Payment record created; awaiting verification |
| `VERIFIED` | Admin confirmed the bank transfer |
| `REJECTED` | UTR/reference could not be verified |

---

## FundingPartnerType

| Value | Meaning |
|---|---|
| `SELF` | MobPae funds from own capital |
| `NBFC` | Non-Banking Financial Company partner |
| `BANK` | Scheduled commercial bank partner |

---

## PayrollCycleType (on Employer)

| Value | Meaning |
|---|---|
| `MONTHLY` | Single payroll per month |
| `BIWEEKLY` | Payroll every two weeks |
| `WEEKLY` | Weekly payroll |

Currently only `MONTHLY` is fully supported in the recovery date logic.

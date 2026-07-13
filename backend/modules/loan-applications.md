# Module: loan-applications

The core module. Every salary advance goes through this module.

**Controller:** `src/loan-applications/loan-applications.controller.ts`  
**Service:** `src/loan-applications/loan-applications.service.ts`

---

## Endpoints

| Method | Path | Role | Description |
|---|---|---|---|
| POST | `/loan-applications` | EMPLOYEE | Submit new advance request |
| GET | `/loan-applications/preview` | EMPLOYEE | Preview repayment before submitting |
| GET | `/loan-applications/eligibility` | EMPLOYEE | Check if employee is eligible |
| GET | `/loan-applications/my` | EMPLOYEE | Get own applications |
| GET | `/loan-applications/my/:id` | EMPLOYEE | Get one own application |
| POST | `/loan-applications/my/:id/cancel` | EMPLOYEE | Cancel a submitted application |
| GET | `/loan-applications/employer` | EMPLOYER | Get company's applications |
| GET | `/loan-applications/employer/pending` | EMPLOYER | Get applications awaiting approval |
| POST | `/loan-applications/:id/employer-approve` | EMPLOYER | Approve an application |
| POST | `/loan-applications/:id/employer-reject` | EMPLOYER | Reject an application |
| POST | `/loan-applications/bulk-action` | EMPLOYER | Bulk approve/reject |
| GET | `/loan-applications` | ADMIN | Get all applications (paginated) |
| GET | `/loan-applications/employee/:employeeId` | ADMIN | Get one employee's applications |
| GET | `/loan-applications/:id` | ADMIN, EMPLOYER | Get full application detail |
| POST | `/loan-applications/:id/admin-approve` | ADMIN | Admin final approval |
| POST | `/loan-applications/:id/admin-reject` | ADMIN | Admin rejection |

---

## Key Service Methods

### `submit(dto, userId)`
1. Fetches employee + employer + product config
2. Calls `EligibilityService.checkEligibility()` — throws if not eligible
3. Calls `PricingService.computeSnapshot()` — freezes all rates
4. Calls `PricingService.resolveRecoveryDate()` — computes due date
5. Creates `LoanApplication` with status `SUBMITTED`
6. Creates `LoanApplicationHistory` entry
7. Sends notification to employee and employer

### `preview(dto, userId)`
Returns the repayment breakdown without creating an application. Used by the employee app before the final confirm step.

### `employerApprove(id, dto, userId)`
1. Validates application belongs to this employer
2. Sets `employerApprovedAmount` (dto amount or requestedAmount if not specified)
3. Calls `PlatformFeesService.ensureFeeForApplication()` — creates the ₹175 fee record
4. Status → `EMPLOYER_APPROVED` (then immediately to `AWAITING_PLATFORM_FEE_PAYMENT` if fee created)
5. Creates history entry, sends notification to employee

### `adminApprove(id, dto, userId)`
1. Validates application is in `READY_FOR_DISBURSAL`
2. Sets `adminApprovedAmount`
3. Records `adminApprovedBy`, `adminApprovedAt`
4. Sends notification to employee

### `findAllForEmployee(userId)`
Returns all applications for the logged-in employee. Each application includes:
- `repayment` (status, totalAmount, dueDate, interestAmount, principalAmount, interestDays, interestRate)
- `disbursal` (status, disbursedAmount, completedAt)

---

## Application Number Format

```
MP-{productCode}-{year}-{8-digit-sequence}
Example: MP-SA-2026-00000001
```

- `MP` = MobPae
- `SA` = Salary Advance (from `LoanProductType`)
- `2026` = submission year
- `00000001` = padded sequence (auto-generated)

---

## Snapshot Fields

At submission time, these fields are frozen from the live product config. They never change even if admin updates the product config later:

| Field | Source |
|---|---|
| `snapshotAnnualInterestRate` | Active `LoanProductConfig.pricingRules.annualInterestRate` |
| `snapshotInterestFreeThreshold` | Computed from salary + platform % |
| `snapshotProcessingFeeRate` | Active config |
| `snapshotGstRate` | Active config |
| `snapshotMaxAdvancePercentage` | Effective % after employer override |
| `snapshotSalaryInHand` | Employee's salary at submission time |
| `snapshotInterestDays` | Days from submission to recovery date |
| `snapshotRecoveryDate` | Next payroll date per employer calendar |

---

## The Three Amounts

| Field | Set by | Meaning |
|---|---|---|
| `requestedAmount` | Employee | What the employee asked for |
| `employerApprovedAmount` | Employer | What employer approved (may differ — employer can reduce or increase within their config) |
| `adminApprovedAmount` | Admin | Final approved amount for disbursal |

The **disbursed amount** lives on the `Disbursal` record, not on `LoanApplication`.

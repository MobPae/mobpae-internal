# Database Schema

**ORM:** Prisma  
**Database:** PostgreSQL  
**Schema file:** `mobpae-backend/prisma/schema.prisma`

All models use UUID primary keys (`@id @default(uuid())`). Timestamps use `createdAt DateTime @default(now())` and `updatedAt DateTime @updatedAt`.

---

## Model: User

The base authentication model. Every login goes through here.

| Field | Type | Description |
|---|---|---|
| `id` | String (UUID) | Primary key |
| `email` | String (unique) | Login email |
| `password` | String | Bcrypt hash |
| `role` | Enum Role | ADMIN / EMPLOYER / EMPLOYEE |
| `isActive` | Boolean | Account enabled (default true) |
| `passwordChanged` | Boolean | True after first password change |
| `lastLogin` | DateTime? | Last successful login |

**Relations:** has one `Employee` or one `Employer` (never both), has many `UserSession`, `Notification`, `AuditLog`, `PasswordResetToken`.

---

## Model: Employer

| Field | Type | Description |
|---|---|---|
| `id` | String | PK |
| `companyName` | String | Legal company name |
| `companyCode` | String (unique) | Short code e.g. LTIM |
| `contactPerson` | String | Primary contact name |
| `email` | String (unique) | Login email (matches User.email) |
| `phone` | String | |
| `payrollDate` | Int | Day of month (e.g. 28) |
| `payrollCutoffDate` | Int | Cutoff day for this cycle (e.g. 21) |
| `cycleType` | Enum | MONTHLY / BIWEEKLY / WEEKLY |
| `status` | Enum EmployerStatus | PENDING / APPROVED / REJECTED / ACTIVE / SUSPENDED |
| `riskStatus` | Enum EmployerRiskStatus | GOOD / WARNING / BLOCKED |
| `userId` | String (unique) | FK to User |

---

## Model: Employee

| Field | Type | Description |
|---|---|---|
| `id` | String | PK |
| `userId` | String? (unique) | FK to User (null until activated) |
| `employerId` | String | FK to Employer |
| `employeeCode` | String | HR code (unique per employer) |
| `name` | String | Full name |
| `email` | String | Email |
| `phone` | String | |
| `salaryInHand` | Decimal | Net monthly salary in ₹ |
| `profilePhotoUrl` | String? | R2 file key |
| `selfieUrl` | String? | R2 file key |
| `selfieStatus` | Enum SelfieStatus | PENDING / VERIFIED / REJECTED |
| `joiningDate` | DateTime? | For tenure calculation |
| `appActivated` | Boolean | True once employee sets password |
| `employmentStatus` | Enum EmployeeStatus | ACTIVE / INACTIVE |

**Unique constraint:** `(employerId, employeeCode)` — same code cannot repeat within a company.

---

## Model: EmployeeBankAccount

One per employee.

| Field | Type | Description |
|---|---|---|
| `employeeId` | String (unique) | FK to Employee |
| `accountHolderName` | String | |
| `accountNumber` | String | |
| `ifscCode` | String | |
| `bankName` | String? | |
| `upiId` | String? | |
| `verified` | Boolean | Admin-verified (default false) |

---

## Model: LoanProduct

Product catalog entry. Currently only `SA` (Salary Advance) is active.

| Field | Type | Description |
|---|---|---|
| `productType` | Enum LoanProductType (unique) | SA / PL / HL / VL / EL / CL |
| `displayName` | String | e.g. "Salary Advance" |
| `isActive` | Boolean | Whether product is available |
| `comingSoon` | Boolean | Show as "coming soon" in app |

---

## Model: LoanProductConfig

Immutable versioned configuration for a product. When you need to change rates, create a new version row; don't edit the existing one.

| Field | Type | Description |
|---|---|---|
| `productId` | String | FK to LoanProduct |
| `versionNumber` | Int | Monotonically increasing version |
| `isActive` | Boolean | Only one version active at a time |
| `effectiveFrom` | DateTime | When this version takes effect |
| `previousVersionId` | String? (unique) | Version chain (singly-linked list) |
| `eligibilityRules` | Json | See below |
| `pricingRules` | Json | See below |
| `operationalRules` | Json | See below |

**eligibilityRules shape:**
```json
{
  "maximumAdvancePercentage": 50,
  "platformAdvancePercentage": 10,
  "platformMaxAdvanceAmount": 5000,
  "hardCeilingPercentage": 50,
  "minimumAdvanceAmount": 1000,
  "minimumSalaryInHand": 15000,
  "minimumTenureMonths": 0,
  "requiresKyc": true,
  "requiresBankAccount": true,
  "requiresActiveSelfie": true,
  "maxRequestsPerCycle": 1,
  "cooldownDays": 0
}
```

**pricingRules shape:**
```json
{
  "annualInterestRate": 36,
  "processingFeeRate": 0,
  "gstRate": 0,
  "platformFeeAmount": 175,
  "platformFeeCurrency": "INR"
}
```

**operationalRules shape:**
```json
{
  "requiresEmployerApproval": true,
  "requiresAdminApproval": true,
  "minDisbursalDays": 0,
  "maxDisbursalDays": 7
}
```

---

## Model: EmployerProductConfig

Per-employer overrides and feature flags.

| Field | Type | Description |
|---|---|---|
| `employerId` | String | FK to Employer |
| `productId` | String | FK to LoanProduct |
| `maximumAdvanceAmountOverride` | Int? | Absolute ₹ cap (admin-granted) |
| `maximumAdvancePercentageOverride` | Decimal? | % override (employer sets, max = hardCeiling) |
| `requiresEmployerApproval` | Boolean | Default true |
| `isEnabled` | Boolean | Product enabled for this employer |

**Unique:** `(employerId, productId)` — one config per employer per product.

---

## Model: LoanLimit

Admin-set per-employee advance ceiling.

| Field | Type | Description |
|---|---|---|
| `employeeId` | String (unique) | FK to Employee |
| `maximumEligibleAmount` | Decimal | Hard cap set by admin |
| `maxRequestsPerCycle` | Int | Default 1 |
| `cooldownDays` | Int | Default 0 |

---

## Model: LoanApplication

The central entity. Every salary advance request is a `LoanApplication`.

| Field | Type | Description |
|---|---|---|
| `applicationNumber` | String (unique) | MP-SA-2026-00000001 |
| `employeeId` | String | |
| `employerId` | String | |
| `productId` | String | |
| `configId` | String | Frozen config version at submission |
| `fundingPartnerId` | String? | NULL = MobPae self-funded |
| `requestedAmount` | Decimal | Employee's ask |
| `employerApprovedAmount` | Decimal? | Employer's decision |
| `adminApprovedAmount` | Decimal? | Admin's final decision |
| `purposeCategory` | Enum | MEDICAL / EDUCATION / HOUSE_RENT / etc. |
| `purposeNote` | String? | Optional free text |
| `status` | Enum LoanApplicationStatus | See status machine |
| `submittedAt` | DateTime | When employee submitted |
| `snapshotAnnualInterestRate` | Decimal | Frozen at submission |
| `snapshotInterestFreeThreshold` | Decimal | Frozen at submission |
| `snapshotProcessingFeeRate` | Decimal | Frozen at submission |
| `snapshotGstRate` | Decimal | Frozen at submission |
| `snapshotMaxAdvancePercentage` | Decimal | Frozen at submission |
| `snapshotSalaryInHand` | Decimal | Frozen at submission |
| `snapshotInterestDays` | Int | Frozen at submission |
| `snapshotRecoveryDate` | DateTime | Frozen at submission |
| `snapshotPayrollCycle` | DateTime? | First day of recovery month |

**Approval tracking:** `employerApprovedBy`, `employerApprovedAt`, `adminApprovedBy`, `adminApprovedAt`, `rejectedBy`, `rejectedAt`, `rejectionReason`.

---

## Model: Disbursal

| Field | Type | Description |
|---|---|---|
| `loanApplicationId` | String (unique) | One disbursal per application |
| `requestedAmount` | Decimal | |
| `approvedAmount` | Decimal | Admin-approved |
| `disbursedAmount` | Decimal? | Actual amount sent |
| `status` | Enum DisbursalStatus | PENDING / PROCESSING / SUCCESS / FAILED / CANCELLED |
| `disbursalAccountNumber` | String? | Frozen bank details |
| `disbursalIfscCode` | String? | Frozen |
| `disbursalBankName` | String? | Frozen |
| `disbursalAccountHolderName` | String? | Frozen |
| `paymentProvider` | Enum | RAZORPAY_PAYOUT / CASHFREE / BANK_TRANSFER / NACH / INTERNAL |
| `providerReference` | String? | Provider's payout ID |
| `retryCount` | Int | Default 0 |
| `initiatedAt` | DateTime? | |
| `completedAt` | DateTime? | |
| `initiatedBy` | String? | Admin userId |

---

## Model: Repayment

| Field | Type | Description |
|---|---|---|
| `loanApplicationId` | String (unique) | One repayment per application |
| `dueDate` | DateTime | From snapshot (payroll date) |
| `paidDate` | DateTime? | When actually paid |
| `status` | Enum RepaymentStatus | SCHEDULED / PAID / OVERDUE |
| `principalAmount` | Decimal | |
| `interestFreeAmount` | Decimal | |
| `interestBearingAmount` | Decimal | |
| `interestAmount` | Decimal | |
| `processingFee` | Decimal | |
| `gstAmount` | Decimal | |
| `totalAmount` | Decimal | What employee owes |
| `interestRate` | Decimal | Annual % used |
| `interestDays` | Int | Days used in calculation |

---

## Model: EmployerSettlement

| Field | Type | Description |
|---|---|---|
| `employerId` | String | |
| `settlementNumber` | String (unique) | MPS-LTIM-202607-0001 |
| `cycleDate` | DateTime | First day of the recovery month |
| `principalAmount` | Decimal | Sum of all principals |
| `interestAmount` | Decimal | Sum of all interest |
| `totalAmount` | Decimal | Total owed |
| `outstandingAmount` | Decimal | Decrements as payments arrive |
| `employeeCount` | Int | Employees included |
| `status` | Enum EmployerSettlementStatus | DRAFT / GENERATED / PARTIALLY_PAID / PAID / OVERDUE / CANCELLED |
| `dueDate` | DateTime | |
| `paidDate` | DateTime? | |

**Unique:** `(employerId, cycleDate)` — one per employer per cycle.

---

## Model: LoanApplicationFee

The ₹175 platform fee charge per advance.

| Field | Type | Description |
|---|---|---|
| `loanApplicationId` | String (unique) | One fee per application |
| `employeeId` | String | |
| `employerId` | String | |
| `feeType` | Enum | PLATFORM_FEE (only type) |
| `amount` | Decimal | ₹175 at creation time |
| `status` | Enum LoanApplicationFeeStatus | PENDING_PAYMENT / PAID / FAILED / EXPIRED / REFUNDED / WAIVED |
| `providerOrderId` | String? | Razorpay order ID |
| `providerPaymentId` | String? | Razorpay payment ID |
| `paidAt` | DateTime? | |
| `waivedAt` | DateTime? | |
| `waivedBy` | String? | Admin userId |

---

## Model: KycDocument

| Field | Type | Description |
|---|---|---|
| `employeeId` | String | |
| `documentType` | Enum | AADHAR / PAN / SALARY_SLIP / OTHER |
| `filePath` | String | R2 file key |
| `status` | Enum KycStatus | PENDING / VERIFIED / REJECTED |
| `rejectionNote` | String? | Reason if rejected |
| `verifiedBy` | String? | Admin userId |

**Unique:** `(employeeId, documentType)` — one of each type per employee.

---

## Model: FundingPartner

Placeholder for future NBFC partnerships. Currently all advances are self-funded (NULL `fundingPartnerId` on LoanApplication).

| Field | Type | Description |
|---|---|---|
| `name` | String | NBFC / bank name |
| `code` | String (unique) | e.g. MOBPAE_SELF, HDFC_NBFC |
| `type` | Enum | SELF / NBFC / BANK |
| `status` | Enum | ACTIVE / INACTIVE |

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

---

## Model: LoanApplicationHistory

Audit trail of every status change on a `LoanApplication`. One row per transition.

| Field | Type | Description |
|---|---|---|
| `loanApplicationId` | String | FK to LoanApplication |
| `previousStatus` | Enum? | Status before the change (null on first entry) |
| `newStatus` | Enum | Status after the change |
| `changedBy` | String? | userId of who triggered the change |
| `actorRole` | String? | Role of actor (EMPLOYEE / EMPLOYER / ADMIN / SYSTEM) |
| `remarks` | String? | Optional reason or note |
| `createdAt` | DateTime | When the transition happened |

---

## Model: SettlementLineItem

One row per employee repayment included in an `EmployerSettlement`. All amounts and identifiers are frozen at settlement creation time.

| Field | Type | Description |
|---|---|---|
| `settlementId` | String | FK to EmployerSettlement |
| `repaymentId` | String (unique) | FK to Repayment — each repayment in at most one settlement |
| `loanApplicationId` | String | FK to LoanApplication |
| `employeeId` | String | FK to Employee |
| `employeeCode` | String | Frozen at settlement time |
| `employeeName` | String | Frozen at settlement time |
| `loanApplicationNumber` | String | Frozen at settlement time |
| `principalAmount` | Decimal | Frozen |
| `interestAmount` | Decimal | Frozen (default 0) |
| `processingFee` | Decimal | Frozen (default 0) |
| `gstAmount` | Decimal | Frozen (default 0) |
| `totalDeductionAmount` | Decimal | Total employer owes for this employee |
| `status` | Enum SettlementLineItemStatus | INCLUDED / EXCLUDED / DISPUTED |
| `remarks` | String? | Optional note |

---

## Model: SettlementPayment

Records one payment event against an `EmployerSettlement`. Multiple payments are allowed (partial payments).

| Field | Type | Description |
|---|---|---|
| `settlementId` | String | FK to EmployerSettlement |
| `amount` | Decimal | Amount paid in this tranche |
| `paymentMethod` | Enum SettlementPaymentMethod | NEFT / RTGS / IMPS / UPI / NACH / CHEQUE |
| `paymentReference` | String? | MobPae-assigned reference |
| `bankReference` | String? | Employer's UTR/NEFT reference |
| `transactionDate` | DateTime | Date of transaction |
| `receivedDate` | DateTime? | Date MobPae received the funds |
| `verifiedBy` | String? | Admin userId who verified |
| `verifiedAt` | DateTime? | |
| `status` | Enum SettlementPaymentStatus | PENDING / VERIFIED / REJECTED |
| `remarks` | String? | |

---

## Model: Notification

In-app notifications stored per user. Read via `GET /notifications/my`.

| Field | Type | Description |
|---|---|---|
| `userId` | String | FK to User |
| `title` | String | Short notification title |
| `message` | String | Full notification body |
| `type` | Enum NotificationType | SYSTEM / APPLICATION_SUBMITTED / FEE_PAID / etc. |
| `isRead` | Boolean | Default false |
| `createdAt` | DateTime | |

---

## Model: AuditLog

Immutable log of every admin action. Never edited, only appended.

| Field | Type | Description |
|---|---|---|
| `userId` | String? | Who performed the action (null for system actions) |
| `action` | String | e.g. DISBURSAL_CREATED, KYC_VERIFIED, EMPLOYER_SUSPENDED |
| `entityType` | String | e.g. Disbursal, KycDocument, Employer |
| `entityId` | String | UUID of the affected record |
| `oldValue` | Json? | State before the change |
| `newValue` | Json? | State after the change |
| `createdAt` | DateTime | |

---

## Model: UserSession

Stores refresh tokens. One row per active device/session per user.

| Field | Type | Description |
|---|---|---|
| `userId` | String | FK to User |
| `refreshToken` | String | Hashed refresh token |
| `deviceInfo` | String? | User agent string |
| `ipAddress` | String? | IP at session creation |
| `isActive` | Boolean | False = logged out / rotated |
| `createdAt` | DateTime | |
| `updatedAt` | DateTime | Updated on token rotation |

---

## Model: PasswordResetToken

Short-lived token for the "forgot password" flow. Single-use.

| Field | Type | Description |
|---|---|---|
| `userId` | String | FK to User |
| `tokenHash` | String | Hashed token value |
| `tokenSelector` | String (unique) | URL-safe selector used to look up the record |
| `expiresAt` | DateTime | 1 hour from creation |
| `usedAt` | DateTime? | Set when token is consumed |
| `createdAt` | DateTime | |

**Usage:** reset link = `{FRONTEND_URL}/reset-password?token={selector}:{rawToken}`

---

## Model: PaymentOrder

Unified Razorpay order record. Used for both platform fees and (legacy) membership payments.

| Field | Type | Description |
|---|---|---|
| `provider` | Enum PaymentProvider | RAZORPAY (only value currently) |
| `providerOrderId` | String (unique) | Razorpay's `order_xxxx` ID |
| `amount` | Int | Amount in paise (₹ × 100) |
| `currency` | String | INR |
| `employeeId` | String | FK to Employee |
| `purpose` | Enum PaymentOrderPurpose | PLATFORM_FEE / MEMBERSHIP |
| `planKey` | String? | FK to MembershipPlanConfig (membership only) |
| `loanApplicationFeeId` | String? | FK to LoanApplicationFee (platform fee only) |
| `couponCode` | String? | Coupon applied (membership only) |
| `discountAmount` | Int | Discount in paise (default 0) |
| `status` | Enum PaymentOrderStatus | CREATED / PAID / FAILED / EXPIRED |
| `notes` | Json? | Arbitrary metadata |
| `expiresAt` | DateTime | Order expiry (15 min from creation) |

**Relations:** has many `PaymentEvent`, optionally linked to `Membership` or `LoanApplicationFee`.

---

## Model: PaymentEvent

Every webhook event from Razorpay for a `PaymentOrder`. Immutable event log.

| Field | Type | Description |
|---|---|---|
| `orderId` | String | FK to PaymentOrder |
| `providerPaymentId` | String? | Razorpay `pay_xxxx` ID |
| `providerSignature` | String? | HMAC signature from webhook/verify |
| `eventType` | String | e.g. payment.captured, payment.failed |
| `source` | String | WEBHOOK or VERIFY_PAYMENT (client-side) |
| `status` | String | Razorpay payment status |
| `method` | String? | upi / card / netbanking |
| `errorCode` | String? | Razorpay error code if failed |
| `errorDescription` | String? | Human-readable error |
| `rawPayload` | Json? | Full Razorpay webhook payload |
| `capturedAt` | DateTime? | When Razorpay captured the payment |
| `createdAt` | DateTime | |

---

## Model: EmployerEnquiry

Leads captured from the MobPae website's "Request a Demo" form. Pre-onboarding record.

| Field | Type | Description |
|---|---|---|
| `companyName` | String | |
| `contactPerson` | String | |
| `email` | String | |
| `phone` | String | |
| `employeeCount` | Int? | Approximate headcount |
| `status` | Enum EmployerEnquiryStatus | NEW / CONTACTED / CONVERTED / REJECTED |
| `remarks` | String? | Internal notes |
| `employerId` | String? (unique) | Set when enquiry is converted to a live employer |

---

## Model: Setting

Key-value store for system configuration. Admin-editable.

| Field | Type | Description |
|---|---|---|
| `key` | String (unique) | Setting name (e.g. PLATFORM_FEE_CONFIG) |
| `value` | String | JSON-encoded value |

**Known keys:**
- `PLATFORM_FEE_CONFIG` → `{ "amount": 175, "currency": "INR" }`

---

## Model: AppInformation

Content blocks shown in the employee app (FAQ, help text, legal content).

| Field | Type | Description |
|---|---|---|
| `type` | Enum AppInfoType (unique) | FAQ / HELP / TERMS / PRIVACY / etc. |
| `title` | String | Display title |
| `content` | Text | Markdown or plain text content |
| `version` | String? | Content version label |
| `isActive` | Boolean | Whether to show this content |

---

## Model: Membership *(Deprecated)*

Legacy monthly subscription model, replaced by the platform fee. Still in schema for historical data; no new records are created.

| Field | Type | Description |
|---|---|---|
| `employeeId` | String (unique) | FK to Employee |
| `planKey` | String | Membership plan identifier |
| `planType` | String | MONTHLY / ANNUAL |
| `planName` | String | Display name |
| `amount` | Decimal | Plan price |
| `amountPaid` | Decimal? | Actual amount collected |
| `startDate` | DateTime | |
| `endDate` | DateTime | |
| `status` | Enum MembershipStatus | PENDING / ACTIVE / EXPIRED / CANCELLED |
| `couponCode` | String? | Applied coupon |
| `discountAmount` | Decimal? | |
| `paymentOrderId` | String? (unique) | FK to PaymentOrder |
| `verifiedBy` | String? | Admin userId |

---

## Model: MembershipPlanConfig *(Deprecated)*

Configuration for legacy membership plans. Not used for new signups.

| Field | Type | Description |
|---|---|---|
| `planKey` | String (unique) | e.g. MONTHLY_99 |
| `planName` | String | |
| `amount` | Decimal | Price |
| `validityDays` | Int | Duration |
| `billingLabel` | String | e.g. "per month" |
| `perMonthLabel` | String? | Annualized monthly cost label |
| `isPreferred` | Boolean | Highlight as recommended |
| `isActive` | Boolean | |
| `sortOrder` | Int | Display order |

---

## Model: MembershipCoupon *(Deprecated)*

Discount coupons for legacy membership payments.

| Field | Type | Description |
|---|---|---|
| `code` | String (unique) | Coupon code |
| `discountAmount` | Decimal | Fixed ₹ discount |
| `isActive` | Boolean | |
| `validTill` | DateTime? | Expiry date (null = no expiry) |
| `usageLimit` | Int? | Max total uses (null = unlimited) |
| `usedCount` | Int | Times used so far |

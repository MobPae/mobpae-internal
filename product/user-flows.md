# User Flows

---

## Flow 1: Employer Onboarding

```
MobPae Admin
  └─ Creates employer account (company name, email, payroll calendar)
  └─ Sets employer status → APPROVED
  └─ Sends login credentials to employer contact

Employer
  └─ Logs in to employer portal
  └─ Uploads employees (CSV or manually)
  └─ Configures advance % override (optional)
  └─ Activates employees
```

---

## Flow 2: Employee Onboarding

```
Employer Portal
  └─ Adds employee (name, email, employeeCode, salary, joiningDate)
  └─ Employee receives activation email with login link

Employee App
  └─ Sets password
  └─ Uploads KYC (Aadhaar, PAN, Salary Slip)
  └─ Uploads bank account details (account number, IFSC)
  └─ Takes selfie

MobPae Admin Panel
  └─ Reviews KYC documents → Verified / Rejected
  └─ Reviews bank account → Verified / Rejected
  └─ Reviews selfie → Verified / Rejected
  └─ Sets LoanLimit (max advance amount for this employee)

Employee App
  └─ App state refreshes → Employee is fully onboarded
  └─ Dashboard shows advance eligibility
```

---

## Flow 3: Salary Advance Request (Full Happy Path)

This is the core flow. Each step is a distinct system state.

```
STEP 1 — Employee submits request
  Employee App
    └─ Selects advance amount (within eligibility limit)
    └─ Selects purpose category (Medical, Rent, Emergency, etc.)
    └─ Reviews repayment preview (principal + interest + total)
    └─ Confirms → POST /loan-applications
  
  System
    └─ Creates LoanApplication with status: SUBMITTED
    └─ Snapshots interest rate, processing fee, interest days, recovery date
    └─ Generates application number: MP-SA-2026-XXXXXXXX
    └─ Sends notification to employee + employer

---

STEP 2 — Employer approves
  Employer Portal
    └─ Sees new application in pending list
    └─ Reviews: employee name, requested amount, purpose
    └─ Can optionally override the amount (within their configured ceiling)
    └─ Clicks Approve → POST /loan-applications/:id/employer-approve
  
  System
    └─ Status → EMPLOYER_APPROVED
    └─ Creates LoanApplicationFee record (₹175, status: PENDING_PAYMENT)
    └─ Sends notification to employee: "Your employer approved. Pay ₹175 to proceed."

  [If employer rejects]
    └─ Status → EMPLOYER_REJECTED
    └─ Flow ends. Employee sees rejection reason.

---

STEP 3 — Employee pays platform fee (₹175)
  Employee App
    └─ Sees "Awaiting Platform Fee Payment" status
    └─ Taps "Pay ₹175" → GET /platform-fees/config (gets Razorpay key + amount)
    └─ POST /platform-fees/loan-applications/:id/initiate-payment
       System creates Razorpay Order, returns order_id
    └─ Razorpay checkout opens on device
    └─ Employee completes payment

  Razorpay
    └─ Sends webhook: payment.captured

  System (webhook handler)
    └─ Verifies Razorpay signature
    └─ Updates LoanApplicationFee status → PAID
    └─ Status → READY_FOR_DISBURSAL
    └─ Sends notification: "Fee paid. MobPae is reviewing your advance."

  [If employee can't pay, admin can waive]
    └─ Admin: POST /platform-fees/:id/waive
    └─ Status → READY_FOR_DISBURSAL directly

---

STEP 4 — Admin reviews and approves
  Admin Panel
    └─ Sees application in ready-for-disbursal queue
    └─ Reviews: employee profile, KYC, bank account, employer, amounts
    └─ Clicks Approve → POST /loan-applications/:id/admin-approve
       Optionally overrides amount (adminApprovedAmount)
  
  System
    └─ Status unchanged (stays READY_FOR_DISBURSAL for disbursal step)
    └─ Admin approval recorded (adminApprovedBy, adminApprovedAt)

  [If admin rejects]
    └─ Status → ADMIN_REJECTED
    └─ Employee notified. Flow ends.

---

STEP 5 — Admin initiates disbursal
  Admin Panel
    └─ POST /disbursals → creates Disbursal record (status: PENDING)
    └─ POST /disbursals/:id/disburse → triggers actual payout
  
  System
    └─ Reads frozen bank account details from Disbursal record
    └─ Calls payment provider (Razorpay Payout / NEFT / NACH)
    └─ On success:
       └─ Disbursal status → SUCCESS
       └─ LoanApplication status → DISBURSED
       └─ Creates Repayment record (principal, interest, total, due date)
       └─ LoanApplication status → REPAYMENT_SCHEDULED
    └─ Employee notified: "₹X disbursed to your account."

---

STEP 6 — Repayment
  On payroll date
    └─ Employer deducts repayment from employee salary
    └─ Employer pays MobPae via bank transfer / NEFT / RTGS

  Admin Panel
    └─ Generates EmployerSettlement for the payroll cycle
       (one settlement per employer per month, containing all employees)
    └─ Sends settlement report to employer
    └─ When payment received: POST /employer-settlements/:id/mark-paid
    └─ POST /repayments/:id/pay → marks individual repayment as PAID
    └─ LoanApplication status → REPAID
    └─ Employee notified: "Your advance has been fully repaid."
```

---

## Flow 4: Employer Bulk Approval

```
Employer Portal
  └─ Selects multiple applications (checkboxes)
  └─ POST /loan-applications/bulk-action
     Body: { applicationIds: [...], action: "approve" | "reject", remarks }
  
  System
    └─ Processes each application individually
    └─ Returns per-application result (success / failed with reason)
```

---

## Flow 5: Settlement Cycle (Monthly)

```
At month end / payroll date:

Admin Panel
  └─ Identifies all REPAYMENT_SCHEDULED advances due this cycle
  └─ Generates EmployerSettlement (one per employer)
     └─ Creates SettlementLineItem for each repayment
     └─ Freezes all amounts at generation time
  └─ POST /employer-settlements/:id/send-report
     └─ Sends PDF settlement report to employer email

Employer
  └─ Deducts repayment amounts from employee salaries
  └─ Transfers total settlement amount to MobPae bank account
  └─ Shares payment reference (UTR/NEFT)

Admin Panel
  └─ Records payment: SettlementPayment (amount, bankReference, date)
  └─ POST /employer-settlements/:id/mark-paid
  └─ Marks individual Repayments as PAID
  └─ LoanApplications move to REPAID
```

---

## Flow 6: KYC Rejection and Resubmission

```
Employee App
  └─ Uploads KYC document

Admin Panel
  └─ Reviews → POST /kyc-documents/:id/reject (with rejection note)
  
  System
    └─ KycDocument status → REJECTED

Employee App
  └─ Sees rejection reason on KYC screen
  └─ Re-uploads corrected document (replaces old record)
  └─ Admin reviews again
```

---

## Application Status State Machine

```
SUBMITTED
  ├─→ EMPLOYER_APPROVED
  │     └─→ AWAITING_PLATFORM_FEE_PAYMENT  (if fee not yet created)
  │           └─→ READY_FOR_DISBURSAL (fee PAID or WAIVED)
  │                 └─→ DISBURSED
  │                       └─→ REPAYMENT_SCHEDULED
  │                             └─→ REPAID ✅
  │                 └─→ ADMIN_REJECTED ❌
  ├─→ EMPLOYER_REJECTED ❌
  ├─→ CANCELLED (by employee, only from SUBMITTED)
  └─→ EXPIRED (system, after configured timeout)
```

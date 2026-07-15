# API Reference — Admin Panel

All requests require `Authorization: Bearer <accessToken>` (role: ADMIN).  
Base URL: `https://api.mobpae.com`

---

## Auth

Same as employee app. Admin login: `POST /auth/login`.

---

## Employers

### GET /employers
Paginated list of all employers.
```
Query: ?page=1&limit=20&search=LTIM&status=ACTIVE
```
```json
Response: {
  "data": [...],
  "total": 42,
  "page": 1,
  "limit": 20
}
```

### GET /employers/:id
Full employer detail including product configs.
```json
Response: {
  "id": "uuid",
  "companyName": "LTIMindtree",
  "companyCode": "LTIM",
  "status": "ACTIVE",
  "riskStatus": "GOOD",
  "payrollDate": 28,
  "payrollCutoffDate": 21,
  "productConfigs": [
    {
      "productType": "SA",
      "isEnabled": true,
      "maximumAdvanceAmountOverride": 10000,
      "maximumAdvancePercentageOverride": null,
      "requiresEmployerApproval": true
    }
  ]
}
```

### POST /employers
Create a new employer. Sends activation email.
```json
Request: {
  "companyName": "string",
  "companyCode": "string (unique, e.g. LTIM)",
  "contactPerson": "string",
  "email": "string",
  "phone": "string",
  "payrollDate": 28,
  "payrollCutoffDate": 21
}
```

### PATCH /employers/:id
Update employer details or status.
```json
Request: { "status": "ACTIVE" | "SUSPENDED", "companyName": "string (optional)" }
```

### PATCH /employers/:id/product-config
Update the employer's product config (override advance limit, enable/disable product).
```json
Request: {
  "productType": "SA",
  "maximumAdvanceAmountOverride": 10000,
  "maximumAdvancePercentageOverride": null,
  "isEnabled": true
}
```

### GET /employers/:id/members
List all active and suspended team members for any employer.
```json
Response: [
  {
    "id": "uuid",
    "role": "OWNER",
    "status": "ACTIVE",
    "officeCode": null,
    "joinedAt": "2024-01-01T00:00:00Z",
    "user": { "id": "uuid", "email": "owner@ltim.com", "lastLogin": "2024-07-15T09:00:00Z" }
  }
]
```

### GET /employers/:id/invites
List pending invites for any employer.
```json
Response: [
  {
    "id": "uuid",
    "email": "user@ltim.com",
    "role": "HR",
    "expiresAt": "2024-07-18T10:00:00Z",
    "resendCount": 0,
    "lastSentAt": "2024-07-15T10:00:00Z",
    "createdAt": "2024-07-15T10:00:00Z",
    "invitedByUser": { "email": "owner@ltim.com" }
  }
]
```

---

## Employees

### GET /employees
Paginated list of all employees across all employers.
```
Query: ?page=1&limit=20&search=Ravi&employerId=uuid&status=ACTIVE
```

### GET /employees/:id/kyc-status
KYC summary for one employee.
```json
Response: {
  "overall": "VERIFIED",
  "documents": {
    "AADHAR": "VERIFIED",
    "PAN": "VERIFIED",
    "SALARY_SLIP": "PENDING"
  },
  "bankAccount": "VERIFIED",
  "selfie": "VERIFIED"
}
```

### POST /employees/:id/selfie/verify
Approve an employee's selfie.
```json
Response: { "selfieStatus": "VERIFIED" }
```

### POST /employees/:id/selfie/reject
```json
Request:  { "note": "string" }
Response: { "selfieStatus": "REJECTED" }
```

---

## Loan Limits

### GET /loan-limits/:employeeId
Get the admin-set advance limit for an employee.
```json
Response: {
  "employeeId": "uuid",
  "maximumEligibleAmount": 5000,
  "maxRequestsPerCycle": 1,
  "cooldownDays": 0
}
```

### POST /loan-limits
Set or update the loan limit for an employee.
```json
Request: {
  "employeeId": "uuid",
  "maximumEligibleAmount": 5000,
  "maxRequestsPerCycle": 1,
  "cooldownDays": 0
}
```

---

## KYC Documents

### GET /kyc-documents/employee/:employeeId
All KYC docs for a specific employee.
```json
Response: [
  {
    "id": "uuid",
    "documentType": "AADHAR",
    "filePath": "employees/kyc/uuid/AADHAR/file.jpg",
    "status": "PENDING",
    "rejectionNote": null
  }
]
```

### POST /kyc-documents/:id/verify
```json
Response: { "status": "VERIFIED" }
```

### POST /kyc-documents/:id/reject
```json
Request:  { "note": "Wrong document uploaded" }
Response: { "status": "REJECTED", "rejectionNote": "Wrong document uploaded" }
```

---

## Bank Accounts

### GET /bank-accounts/employee/:employeeId
```json
Response: {
  "accountHolderName": "Ravi Kumar",
  "accountNumber": "1234567890",
  "ifscCode": "SBIN0001234",
  "bankName": "State Bank of India",
  "verified": false
}
```

### POST /bank-accounts/:id/verify
Admin confirms the bank account is valid.
```json
Response: { "verified": true }
```

---

## Loan Applications

### GET /loan-applications
Paginated list of all applications across all employers.
```
Query: ?page=1&limit=20&status=SUBMITTED&employerId=uuid&search=MP-SA
```
```json
Response: {
  "data": [
    {
      "id": "uuid",
      "applicationNumber": "MP-SA-2024-00000001",
      "status": "READY_FOR_DISBURSAL",
      "requestedAmount": 5000,
      "adminApprovedAmount": 5000,
      "employee": { "name": "Ravi Kumar", "salaryInHand": 45000 },
      "employer": { "companyName": "LTIMindtree" },
      "repayment": { ...breakdown... },
      "createdAt": "..."
    }
  ],
  "total": 150,
  "page": 1,
  "limit": 20
}
```

### GET /loan-applications/:id
Full detail of one application.
```json
Response: {
  "id": "uuid",
  "applicationNumber": "MP-SA-2024-00000001",
  "status": "READY_FOR_DISBURSAL",
  "requestedAmount": 5000,
  "employerApprovedAmount": 5000,
  "adminApprovedAmount": 5000,
  "employee": { ...full employee... },
  "employer": { ...full employer... },
  "repayment": { ...full breakdown... },
  "disbursal": { "status": "PENDING", "disbursedAmount": null },
  "fee": { "status": "PAID", "amount": 175 },
  "snapshotSalaryInHand": 45000,
  "snapshotAnnualInterestRate": 36,
  "snapshotInterestDays": 14,
  "snapshotRecoveryDate": "2024-02-28T00:00:00Z"
}
```

### GET /loan-applications/employee/:employeeId
All applications for a specific employee.

### POST /loan-applications/:id/admin-approve
Admin approves an application (moves to `READY_FOR_DISBURSAL`).
No body required — approves the employer-approved amount.
```json
Response: { "id": "uuid", "status": "READY_FOR_DISBURSAL" }
```

### POST /loan-applications/:id/admin-reject
```json
Request:  { "reason": "string", "remarks": "string (optional)" }
Response: { "id": "uuid", "status": "ADMIN_REJECTED" }
```

---

## Platform Fees

### GET /platform-fees
Paginated list of all platform fee records.
```
Query: ?page=1&limit=20&status=PAID
```

### GET /platform-fees/:id
Detail of one fee record.

### POST /platform-fees/:id/waive
Waive a fee without requiring payment (moves application to `READY_FOR_DISBURSAL`).
```json
Request:  { "reason": "Promotional waiver" }
Response: { "status": "WAIVED" }
```

---

## Disbursals

### POST /disbursals
Create a disbursal record for an approved application. Freezes bank account details and creates the Repayment record.
```json
Request: { "loanApplicationId": "uuid" }
Response: {
  "id": "uuid",
  "status": "PENDING",
  "approvedAmount": 5000,
  "disbursalAccountNumber": "1234567890",
  "disbursalIfscCode": "SBIN0001234"
}
```

### GET /disbursals
List all disbursals.
```
Query: ?page=1&limit=20&status=PENDING
```

### POST /disbursals/:id/disburse
Mark a disbursal as complete (trigger money transfer).
```json
Response: { "id": "uuid", "status": "SUCCESS", "completedAt": "..." }
```

---

## Repayments

### GET /repayments
All repayments across all employers.
```
Query: ?page=1&limit=20&status=SCHEDULED&employerId=uuid
```

### GET /repayments/:id
Single repayment detail with linked application.

---

## Employer Settlements

### GET /employer-settlements
All settlements.
```
Query: ?page=1&limit=20&status=GENERATED&employerId=uuid
```

### GET /employer-settlements/:id
Full settlement with all line items and payment history.

### POST /employer-settlements/check-risk/:employerId
Evaluate overdue settlements and update employer risk status.
```json
Response: { "riskStatus": "WARNING", "overdueCount": 1, "overdueAmount": 52500 }
```

### POST /employer-settlements/:id/mark-paid
Record that a settlement has been paid.
```json
Request: {
  "amount": 52500,
  "paymentMethod": "NEFT",
  "bankReference": "NEFT20240201123",
  "transactionDate": "2024-02-01T00:00:00Z"
}
Response: { "status": "PAID", "outstandingAmount": 0 }
```

### POST /employer-settlements/:id/send-report
Email settlement report to employer.

---

## Loan Products & Config

### GET /loan-products
List all products.
```json
Response: [
  { "productType": "SA", "displayName": "Salary Advance", "isActive": true, "comingSoon": false }
]
```

### GET /loan-products/SA/config
Active config for a product.

### POST /loan-products/SA/config
Create a new config version. Previous version is deactivated.
```json
Request: {
  "eligibilityRules": {
    "maximumAdvancePercentage": 50,
    "platformAdvancePercentage": 10,
    "platformMaxAdvanceAmount": 5000,
    "minimumAdvanceAmount": 1000,
    "minimumSalaryInHand": 15000,
    "minimumTenureMonths": 0,
    "maxRequestsPerCycle": 1,
    "cooldownDays": 0,
    "requiresKyc": true,
    "requiresBankAccount": true,
    "requiresActiveSelfie": true
  },
  "pricingRules": {
    "annualInterestRate": 36,
    "processingFeeRate": 0,
    "gstRate": 0,
    "platformFeeAmount": 175,
    "platformFeeCurrency": "INR"
  },
  "operationalRules": {
    "requiresEmployerApproval": true,
    "requiresAdminApproval": true
  }
}
```

---

## Files (Signed URLs)

### GET /files/signed-url?key=<r2_key>
Get a 15-minute signed URL to view any private file (KYC docs, selfies, etc.).
```json
Response: { "url": "https://r2.mobpae.com/..." }
```

---

## Settings

### GET /settings/:key
Get a setting by key. Known keys:
- `PLATFORM_FEE_CONFIG` — `{ amount: 175, currency: "INR" }`
- `APP_INFO_*` — various app content blocks

### PUT /settings/:key
Update a setting value.
```json
Request: { "value": <any JSON> }
```

---

## App Information

### GET /app-information
List all content entries.

### POST /app-information
Create a new content entry.
```json
Request: { "type": "FAQ", "content": "markdown string" }
```

### PATCH /app-information/:id
Update a content entry.

### DELETE /app-information/:id
Delete a content entry.

---

## Audit Log

### GET /audit-logs
Paginated audit trail of all admin actions.
```
Query: ?page=1&limit=50&userId=uuid&action=DISBURSAL_CREATED
```
```json
Response: {
  "data": [
    {
      "id": "uuid",
      "userId": "uuid",
      "action": "DISBURSAL_CREATED",
      "entityType": "Disbursal",
      "entityId": "uuid",
      "meta": { "approvedAmount": 5000 },
      "createdAt": "..."
    }
  ]
}
```

---

## Notifications

### GET /notifications/my
### POST /notifications/mark-read
### POST /notifications/mark-all-read

Same structure as employee. Admins receive notifications for new applications, payments, etc.

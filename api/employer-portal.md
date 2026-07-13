# API Reference — Employer Portal

All requests require `Authorization: Bearer <accessToken>` (role: EMPLOYER).  
Base URL: `https://api.mobpae.com`

---

## Auth

Same endpoints as employee app. See [employee-app.md](./employee-app.md#auth).

---

## Employer Profile

### GET /employers/me
```json
Response: {
  "id": "uuid",
  "companyName": "LTIMindtree",
  "companyCode": "LTIM",
  "contactPerson": "Priya Sharma",
  "email": "priya@ltim.com",
  "phone": "9876543210",
  "payrollDate": 28,
  "payrollCutoffDate": 21,
  "cycleType": "MONTHLY",
  "status": "ACTIVE",
  "riskStatus": "GOOD"
}
```

### PATCH /employers/me
Update employer profile.
```json
Request: {
  "contactPerson": "string (optional)",
  "phone": "string (optional)",
  "payrollDate": number,
  "payrollCutoffDate": number
}
```

---

## Employees

### GET /employees/employer
List all employees in the employer's company.
```json
Response: [
  {
    "id": "uuid",
    "employeeCode": "EMP001",
    "name": "Ravi Kumar",
    "email": "ravi@ltim.com",
    "salaryInHand": 45000,
    "employmentStatus": "ACTIVE",
    "appActivated": true,
    "kycStatus": "VERIFIED",
    "bankStatus": "VERIFIED",
    "selfieStatus": "VERIFIED"
  }
]
```

### POST /employees
Create a single employee. Employee gets an activation email.
```json
Request: {
  "employeeCode": "EMP001",
  "name": "Ravi Kumar",
  "email": "ravi@ltim.com",
  "phone": "9876543210",
  "salaryInHand": 45000,
  "joiningDate": "2023-01-01 (optional)"
}
Response: { "id": "uuid", "employeeCode": "EMP001", "appActivated": false }
```

### POST /employees/bulk
Bulk create employees from CSV data.
```json
Request: {
  "employees": [
    { "employeeCode": "EMP001", "name": "...", "email": "...", "phone": "...", "salaryInHand": 0 }
  ]
}
Response: {
  "created": 47,
  "failed": 3,
  "errors": [{ "row": 5, "reason": "Email already exists" }]
}
```

### PATCH /employees/:id
Update employee details (salary, name, etc.).
```json
Request: { "salaryInHand": 50000, "name": "string (optional)", "phone": "string (optional)" }
```

### PATCH /employees/:id/activation
Activate or deactivate a single employee.
```json
Request: { "isActive": true }
```

### PATCH /employees/bulk-activation
Bulk activate or deactivate employees.
```json
Request: { "employeeIds": ["uuid1", "uuid2"], "isActive": false }
```

---

## Loan Applications

### GET /loan-applications/employer
List all loan applications for this employer's employees.
```json
Response: [
  {
    "id": "uuid",
    "applicationNumber": "MP-SA-2024-00000001",
    "status": "SUBMITTED",
    "requestedAmount": 5000,
    "employee": { "name": "Ravi Kumar", "employeeCode": "EMP001" },
    "repayment": { "status": "SCHEDULED", "totalAmount": 5000, "dueDate": "..." },
    "createdAt": "..."
  }
]
```

### GET /loan-applications/employer/pending
Only applications in `SUBMITTED` status awaiting employer action.

### GET /loan-applications/:id
Get full detail of a single application.
```json
Response: {
  "id": "uuid",
  "applicationNumber": "...",
  "status": "SUBMITTED",
  "requestedAmount": 5000,
  "employerApprovedAmount": null,
  "adminApprovedAmount": null,
  "employee": { "name": "...", "email": "...", "salaryInHand": 45000 },
  "repayment": { ...full breakdown... },
  "fee": { "status": "PENDING_PAYMENT", "amount": 175 },
  "submittedAt": "...",
  "snapshotSalaryInHand": 45000,
  "snapshotAnnualInterestRate": 36,
  "snapshotRecoveryDate": "...",
  "snapshotInterestDays": 14
}
```

### POST /loan-applications/:id/employer-approve
Approve an application. No request body needed — employer approves the requested amount as-is.
```json
Response: { "id": "uuid", "status": "EMPLOYER_APPROVED" }
```

Note: If employer wants to approve a different amount, that currently goes through admin. The employer approval is binary (approve at requested amount or reject).

### POST /loan-applications/:id/employer-reject
```json
Request:  { "reason": "string", "remarks": "string (optional)" }
Response: { "id": "uuid", "status": "EMPLOYER_REJECTED" }
```

### POST /loan-applications/bulk-action
Approve or reject up to 50 applications at once.
```json
Request: {
  "ids": ["uuid1", "uuid2"],
  "action": "APPROVE" | "REJECT",
  "remarks": "string (optional)"
}
Response: {
  "processed": 2,
  "failed": 0,
  "results": [{ "id": "uuid1", "status": "EMPLOYER_APPROVED" }]
}
```

---

## Repayments

### GET /repayments/employer
List all repayments for the employer's employees.
```json
Response: [
  {
    "id": "uuid",
    "status": "SCHEDULED",
    "dueDate": "2024-02-28T00:00:00Z",
    "totalAmount": 5000,
    "principalAmount": 5000,
    "interestAmount": 0,
    "employee": { "name": "Ravi Kumar", "employeeCode": "EMP001" },
    "loanApplication": { "applicationNumber": "MP-SA-2024-00000001" }
  }
]
```

---

## Settlements

### GET /employer-settlements/employer
List all settlements for this employer.
```json
Response: [
  {
    "id": "uuid",
    "settlementNumber": "MPS-LTIM-202407-0001",
    "cycleDate": "2024-07-01T00:00:00Z",
    "totalAmount": 52500,
    "outstandingAmount": 52500,
    "status": "GENERATED",
    "dueDate": "2024-07-28T00:00:00Z",
    "employeeCount": 10
  }
]
```

### GET /employer-settlements/employer/summary
Summary stats (total outstanding, count by status, etc.).

### GET /employer-settlements/:id
Full settlement detail with line items.
```json
Response: {
  "id": "uuid",
  "settlementNumber": "MPS-LTIM-202407-0001",
  "principalAmount": 50000,
  "interestAmount": 2500,
  "totalAmount": 52500,
  "outstandingAmount": 0,
  "status": "PAID",
  "lineItems": [
    {
      "employeeName": "Ravi Kumar",
      "employeeCode": "EMP001",
      "loanApplicationNumber": "MP-SA-2024-00000001",
      "principalAmount": 5000,
      "interestAmount": 250,
      "totalDeductionAmount": 5250
    }
  ],
  "payments": [
    { "amount": 52500, "paymentMethod": "NEFT", "bankReference": "NEFT123", "status": "VERIFIED" }
  ]
}
```

### POST /employer-settlements/:id/send-report
Email the settlement report PDF to the employer's email.
```json
Response: { "message": "Report sent to priya@ltim.com" }
```

---

## Notifications

### GET /notifications/my
### POST /notifications/mark-read
### POST /notifications/mark-all-read

Same as employee app.

---

## Settings / App Information

### GET /app-information/settings
Get general platform settings visible to employer (e.g., payroll config).

### GET /app-information/type/:type
Same as employee app.

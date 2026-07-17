# API Reference — Employee App

All requests require `Authorization: Bearer <accessToken>` unless marked Public.  
Base URL: `https://api.mobpae.com` (or `http://localhost:3000` in dev)

---

## Auth

### POST /auth/login  *(Public)*
```json
Request:  { "email": "string", "password": "string" }
Response: {
  "accessToken": "...",
  "refreshToken": "...",
  "user": {
    "id": "uuid",
    "email": "string",
    "role": "EMPLOYEE",
    "employeeId": "uuid",
    "passwordChanged": false,   // false → show forced password gate
    "termsAccepted": false      // false → show T&C gate (checked after pwd gate)
  }
}
```

### POST /auth/refresh  *(Public)*
```json
Request:  { "refreshToken": "string" }
Response: { "accessToken": "...", "refreshToken": "..." }
```

### POST /auth/logout
```json
Request:  { "refreshToken": "string" }
Response: { "message": "Logged out" }
```

### POST /auth/forgot-password  *(Public)*
```json
Request:  { "email": "string" }
Response: { "message": "If account exists, reset email sent" }
```

### POST /auth/reset-password  *(Public)*
```json
Request:  { "token": "string", "newPassword": "string" }
Response: { "message": "Password reset successful" }
```

### POST /auth/change-password
```json
Request:  { "currentPassword": "string", "newPassword": "string" }

// Forced first-time change: returns new tokens so the session continues
Response (forced): {
  "accessToken": "...",
  "refreshToken": "...",
  "termsAccepted": false   // false → show T&C gate next
}

// Voluntary change: invalidates all sessions; user must log in again
Response (voluntary): { "success": true }
```

### POST /auth/accept-terms
Records that the authenticated user has accepted the Terms & Conditions.  
Called once after the forced password change gate passes.
```json
Request:  (no body)
Response: { "success": true }
```

### GET /auth/me
```json
Response: { "id": "uuid", "email": "string", "role": "EMPLOYEE" }
```

---

## Employee Profile

### GET /employees/me
Returns basic employee profile.
```json
Response: {
  "id": "uuid",
  "name": "Ravi Kumar",
  "email": "ravi@ltim.com",
  "phone": "9876543210",
  "salaryInHand": 45000,
  "employmentStatus": "ACTIVE",
  "appActivated": true,
  "employer": { "companyName": "LTIMindtree", "companyCode": "LTIM" }
}
```

### GET /employees/me/app-state
The primary endpoint. Called on every app launch to get full dashboard state.
```json
Response: {
  "employee": { ...profile fields... },
  "eligibility": {
    "isEligible": true,
    "reason": "Eligible for salary advance",
    "maxAdvanceAmount": 5000,
    "advanceLimit": 5000,
    "checks": {
      "kycComplete": true,
      "bankVerified": true,
      "salaryMinimumMet": true,
      "tenureMet": true,
      "noActiveAdvance": true,
      "cooldownPassed": true
    }
  },
  "kycStatus": "VERIFIED",
  "bankStatus": "VERIFIED",
  "activeApplication": null,
  "scheduledRepayment": null,
  "platformFee": {
    "amount": 175,
    "amountPaise": 17500,
    "currency": "INR"
  }
}
```

### GET /employees/me/profile
Profile card (includes photo URLs resolved as signed URLs).

### GET /employees/me/peer-activity
Anonymised list of recent advances from the same company (social proof).
```json
Response: [
  { "displayName": "R***", "amount": 5000, "daysAgo": 3 }
]
```

### POST /employees/profile-photo
Upload profile photo. `multipart/form-data` with field `file` (max 5 MB).
```json
Response: { "profilePhotoUrl": "employees/uploads/uuid/profile.jpg" }
```

---

## KYC Documents

### GET /kyc-documents/my
Returns all KYC documents for the current employee.
```json
Response: [
  {
    "id": "uuid",
    "documentType": "AADHAR",
    "status": "VERIFIED",
    "rejectionNote": null,
    "createdAt": "2024-01-01T00:00:00Z"
  }
]
```

### POST /kyc-documents
Upload a KYC document. `multipart/form-data` with fields `documentType` and `file`.
```json
Request (form): { "documentType": "AADHAR" | "PAN" | "SALARY_SLIP" | "OTHER", "file": <binary> }
Response: { "id": "uuid", "documentType": "AADHAR", "status": "PENDING" }
```

---

## Bank Account

### GET /bank-accounts/my
```json
Response: {
  "id": "uuid",
  "accountHolderName": "Ravi Kumar",
  "accountNumber": "1234567890",
  "ifscCode": "SBIN0001234",
  "bankName": "State Bank of India",
  "upiId": null,
  "verified": false
}
```
Returns `null` if no bank account added yet.

### POST /bank-accounts
Add bank account (replaces existing if already added).
```json
Request: {
  "accountHolderName": "string",
  "accountNumber": "string",
  "ifscCode": "string",
  "bankName": "string",
  "upiId": "string (optional)"
}
Response: { "id": "uuid", "verified": false }
```

---

## Loan Applications

### GET /loan-applications/eligibility
Check eligibility before showing the advance form.
```json
Response: {
  "isEligible": true,
  "reason": "string",
  "maxAdvanceAmount": 5000,
  "advanceLimit": 5000,
  "checks": { ...same as app-state... }
}
```

### GET /loan-applications/preview?amount=5000
Preview repayment breakdown before submitting.
```json
Response: {
  "requestedAmount": 5000,
  "principalAmount": 5000,
  "interestFreeAmount": 5000,
  "interestBearingAmount": 0,
  "interestAmount": 0,
  "processingFee": 0,
  "gstAmount": 0,
  "totalAmount": 5000,
  "interestRate": 36,
  "interestDays": 14,
  "dueDate": "2024-02-28T00:00:00Z"
}
```

### POST /loan-applications
Submit an advance request.
```json
Request: {
  "amount": 5000,
  "purposeCategory": "MEDICAL",
  "purposeNote": "Hospital bills (optional)",
  "remarks": "Urgent (optional)"
}
Response: {
  "id": "uuid",
  "applicationNumber": "MP-SA-2024-00000001",
  "status": "SUBMITTED",
  "requestedAmount": 5000,
  "createdAt": "..."
}
```

### GET /loan-applications/my
Returns all own applications.
```json
Response: [
  {
    "id": "uuid",
    "applicationNumber": "MP-SA-2024-00000001",
    "status": "EMPLOYER_APPROVED",
    "requestedAmount": 5000,
    "repayment": {
      "status": "SCHEDULED",
      "totalAmount": 5000,
      "dueDate": "...",
      "interestAmount": 0,
      "principalAmount": 5000,
      "interestDays": 14,
      "interestRate": 36
    },
    "createdAt": "..."
  }
]
```

### GET /loan-applications/my/:id
Single application detail.

### POST /loan-applications/my/:id/cancel
Cancel a submitted application (only while in `SUBMITTED` status).
```json
Request:  { "remarks": "string (optional)" }
Response: { "id": "uuid", "status": "CANCELLED" }
```

---

## Platform Fee (₹175)

### GET /platform-fees/config
Get current platform fee amount (called before showing fee screen).
```json
Response: {
  "amount": 175,
  "currency": "INR",
  "razorpayKeyId": "rzp_live_xxxx"
}
```

### POST /platform-fees/loan-applications/:loanApplicationId/initiate-payment
Create (or reuse) a Razorpay order for the fee.
```json
Response: {
  "orderId": "order_xxxx",
  "amount": 17500,
  "currency": "INR",
  "key": "rzp_live_xxxx",
  "feeId": "uuid"
}
```

### POST /platform-fees/verify-payment
Called after Razorpay checkout succeeds.
```json
Request: {
  "razorpayOrderId": "order_xxxx",
  "razorpayPaymentId": "pay_xxxx",
  "razorpaySignature": "hmac_signature"
}
Response: { "success": true, "applicationStatus": "READY_FOR_DISBURSAL" }
```

### GET /platform-fees/my
List own fee records.

---

## Notifications

### GET /notifications/my
```json
Response: {
  "data": [
    {
      "id": "uuid",
      "type": "APPLICATION_SUBMITTED",
      "title": "Application Submitted",
      "message": "Your salary advance request has been submitted.",
      "isRead": false,
      "createdAt": "..."
    }
  ],
  "unreadCount": 3
}
```

### POST /notifications/mark-read
```json
Request: { "ids": ["uuid1", "uuid2"] }
```

### POST /notifications/mark-all-read

---

## Files (Signed URLs)

### GET /files/signed-url?key=<r2_key>
Get a temporary signed URL (15 min) to view a private file (e.g., own KYC doc).
```json
Response: { "url": "https://..." }
```

---

## App Information

### GET /app-information/type/:type
Get content by type (e.g., FAQ, help text).
```json
Params: type = FAQ | HELP | TERMS | PRIVACY
Response: { "type": "FAQ", "content": "markdown string" }
```

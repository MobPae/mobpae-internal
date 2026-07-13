# Runbook: Manual Disbursal

How to disburse an approved salary advance to an employee's bank account.

**Who:** Admin  
**When:** Application is in `READY_FOR_DISBURSAL` status (employer approved + platform fee paid + admin approved)

---

## Prerequisites

Before disbursing, verify:
- Application status is `READY_FOR_DISBURSAL`
- Employee bank account is verified (`bank.verified = true`)
- You have the employee's bank account details on screen

---

## Steps

### 1. Find the application

In the admin panel: **Salary Requests** → filter by status `READY_FOR_DISBURSAL`.

Or via API:
```
GET /loan-applications?status=READY_FOR_DISBURSAL
```

### 2. Review the application

Click the application to open the detail drawer. Verify:
- Employee name and company
- Requested amount
- Admin-approved amount
- Bank account details (account number, IFSC, name)
- Snapshot interest rate and recovery date

### 3. Create the disbursal record

In the admin panel: click **"Create Disbursal"** in the application drawer.

Or via API:
```json
POST /disbursals
{ "loanApplicationId": "uuid" }
```

This:
- Freezes the bank account details on the disbursal record
- Creates the `Repayment` record with the computed breakdown
- Moves application status → `DISBURSED` then `REPAYMENT_SCHEDULED`

### 4. Transfer the money

Currently: transfer the money manually via your bank's NEFT/RTGS portal using the frozen bank details shown on the disbursal record.

Then in the admin panel: click **"Mark as Disbursed"**.

Or via API:
```
POST /disbursals/:id/disburse
```

This marks `Disbursal.status = SUCCESS` and sends a notification to the employee.

### 5. Confirm

Application status should now be `REPAYMENT_SCHEDULED`. Employee will see "Active Request" on their dashboard.

---

## If Disbursal Fails

If the bank transfer is rejected (wrong account number, account closed, etc.):

1. Contact the employee to get corrected bank details
2. Employee updates their bank account via the app
3. Admin verifies the new bank account
4. Create a new disbursal — the old one can be marked `FAILED` or `CANCELLED`
5. Re-disburse to the new account

---

## Notes

- Each application can only have one disbursal record
- Bank account details are frozen at disbursal creation time — if the employee changes their bank after this point, the disbursal still goes to the old account
- The `retryCount` field on Disbursal tracks failed attempts

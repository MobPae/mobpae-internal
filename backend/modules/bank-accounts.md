# Module: bank-accounts

Handles employee bank account setup and admin verification.

**Controller:** `src/bank-accounts/bank-accounts.controller.ts`

---

## Endpoints

| Method | Path | Role | Description |
|---|---|---|---|
| POST | `/bank-accounts` | EMPLOYEE | Submit bank account details |
| POST | `/bank-accounts/employee/:id/upi` | EMPLOYEE | Add UPI ID |
| GET | `/bank-accounts/my` | EMPLOYEE | Get own bank account |
| GET | `/bank-accounts/employee/:id` | ADMIN, EMPLOYEE | Get by employee |
| GET | `/bank-accounts` | ADMIN | List all (paginated) |
| GET | `/bank-accounts/pending-by-employer` | ADMIN | Unverified accounts by employer |
| GET | `/bank-accounts/pending-by-employer/:id` | ADMIN | For specific employer |
| POST | `/bank-accounts/:id/verify` | ADMIN | Mark as verified |
| POST | `/bank-accounts/:id/reject` | ADMIN | Reject with reason |

---

## Fields

| Field | Description |
|---|---|
| `accountHolderName` | Full name as on bank account |
| `accountNumber` | Bank account number |
| `ifscCode` | IFSC code of the branch |
| `bankName` | Bank name (optional, for display) |
| `upiId` | UPI ID (optional) |
| `verified` | Boolean — true after admin verification |

One bank account per employee (`@unique` on `employeeId`). Employee submits once; admin verifies. If rejected, employee must resubmit.

---

## Why Manual Verification?

Currently bank accounts are verified manually by admin (checking account number format, IFSC validity). Automated penny drop verification (sending ₹1 to confirm account) is a planned future enhancement.

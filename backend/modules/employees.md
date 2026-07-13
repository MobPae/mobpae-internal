# Module: employees

Manages employee accounts, onboarding, selfie verification, and profile.

**Controller:** `src/employees/employees.controller.ts`

---

## Endpoints

| Method | Path | Role | Description |
|---|---|---|---|
| POST | `/employees` | EMPLOYER | Create a single employee |
| POST | `/employees/bulk` | EMPLOYER | Bulk create via CSV |
| PATCH | `/employees/bulk-activation` | EMPLOYER | Bulk activate/deactivate |
| PATCH | `/employees/:id/activation` | EMPLOYER | Activate or deactivate one employee |
| PATCH | `/employees/:id` | EMPLOYER | Update employee details |
| GET | `/employees/employer` | EMPLOYER | List own company employees |
| GET | `/employees` | ADMIN | List all employees (paginated) |
| GET | `/employees/me` | EMPLOYEE | Get own profile |
| GET | `/employees/me/app-state` | EMPLOYEE | Full app state (eligibility, active application, repayment) |
| GET | `/employees/me/profile` | EMPLOYEE | Profile card |
| GET | `/employees/me/peer-activity` | EMPLOYEE | Anonymised company activity |
| POST | `/employees/profile-photo` | EMPLOYEE | Upload profile photo |
| POST | `/employees/selfie` | EMPLOYEE | Upload selfie |
| POST | `/employees/:id/selfie/verify` | ADMIN | Verify employee selfie |
| POST | `/employees/:id/selfie/reject` | ADMIN | Reject employee selfie |
| GET | `/employees/:id/kyc-status` | ADMIN, EMPLOYEE | Get KYC status summary |

---

## Employee App State (`GET /employees/me/app-state`)

The most important endpoint for the employee app. Returns everything needed to render the dashboard in a single call:

```json
{
  "employee": { ...profile fields... },
  "eligibility": {
    "isEligible": boolean,
    "reason": string,
    "maxAdvanceAmount": number,
    "advanceLimit": number,
    "checks": {
      "kycComplete": boolean,
      "bankVerified": boolean,
      "selfieVerified": boolean,
      "salaryMinimumMet": boolean,
      "tenureMet": boolean,
      "noActiveAdvance": boolean,
      "cooldownPassed": boolean
    }
  },
  "kycStatus": "VERIFIED | PENDING | REJECTED | NOT_SUBMITTED",
  "bankStatus": "VERIFIED | PENDING | NOT_ADDED",
  "selfieStatus": "VERIFIED | PENDING | REJECTED",
  "activeApplication": { ...full loan application if exists... },
  "scheduledRepayment": { ...if any... },
  "platformFee": { "amount": 175, "amountPaise": 17500, ... }
}
```

---

## Onboarding Sequence

Employees must complete these steps before they can request an advance:

1. **Activate account** — set password from email link
2. **Upload KYC** — Aadhaar, PAN, Salary Slip
3. **Add bank account** — account number + IFSC
4. **Upload selfie** — for identity verification
5. **Admin verifies KYC, bank, and selfie**
6. **Admin sets LoanLimit** — max advance amount for this employee

The `app-state` endpoint tracks which steps are complete.

---

## Selfie

The selfie is separate from KYC documents. It is used for identity verification (face matching). Status values: `PENDING`, `VERIFIED`, `REJECTED`. Admin verifies selfies from the admin panel (compared visually with Aadhaar photo if needed).

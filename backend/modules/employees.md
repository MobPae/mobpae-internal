# Module: employees

Manages employee accounts, onboarding, and profile.

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
      "salaryMinimumMet": boolean,
      "tenureMet": boolean,
      "noActiveAdvance": boolean,
      "cooldownPassed": boolean
    }
  },
  "kycStatus": "VERIFIED | PENDING | REJECTED | NOT_SUBMITTED",
  "bankStatus": "VERIFIED | PENDING | NOT_ADDED",
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
4. **Admin verifies KYC and bank**
5. **Admin sets LoanLimit** — max advance amount for this employee

The `app-state` endpoint tracks which steps are complete.

# Module: employers

Manages employer accounts, status, and product configuration.

**Controller:** `src/employers/employers.controller.ts`

---

## Endpoints

| Method | Path | Role | Description |
|---|---|---|---|
| GET | `/employers` | ADMIN | List all employers (paginated) |
| GET | `/employers/:id` | ADMIN | Get one employer |
| POST | `/employers` | ADMIN | Create employer |
| PATCH | `/employers/:id` | ADMIN | Update employer details |
| POST | `/employers/:id/approve` | ADMIN | Approve employer |
| POST | `/employers/:id/reject` | ADMIN | Reject employer |
| POST | `/employers/:id/suspend` | ADMIN | Suspend employer |

---

## Employer Status Flow

```
PENDING → APPROVED → ACTIVE ✅
        → REJECTED ❌
ACTIVE  → SUSPENDED (admin can suspend for non-payment etc.)
```

---

## Employer Data

| Field | Description |
|---|---|
| `companyName` | Legal company name |
| `companyCode` | Short unique code (e.g. LTIM, WIPRO) |
| `contactPerson` | HR/finance manager name |
| `email` | Login email |
| `phone` | Contact phone |
| `payrollDate` | Day of month salary is paid (e.g. 28) |
| `payrollCutoffDate` | Last day to submit and recover in current cycle (e.g. 21) |
| `cycleType` | MONTHLY (default), BIWEEKLY, WEEKLY |
| `riskStatus` | GOOD / WARNING / BLOCKED — updated by admin based on settlement behaviour |

---

## Employer Product Config

Each employer can have per-product overrides via `EmployerProductConfig`:

| Field | Description |
|---|---|
| `maximumAdvanceAmountOverride` | Absolute ₹ cap override (admin-granted special case) |
| `maximumAdvancePercentageOverride` | % of salary override (employer sets; must be ≤ hardCeiling) |
| `requiresEmployerApproval` | Whether employer must approve before admin sees it |
| `isEnabled` | Whether this product is enabled for this employer |

See `src/employer-product-configs/` module for CRUD endpoints.

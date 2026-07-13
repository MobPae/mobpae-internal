# Module: repayments

Tracks repayment records. Each advance has exactly one repayment record.

**Controller:** `src/repayments/repayments.controller.ts`

---

## Endpoints

| Method | Path | Role | Description |
|---|---|---|---|
| GET | `/repayments` | ADMIN | List all repayments (paginated) |
| GET | `/repayments/employer` | EMPLOYER | Own company's repayments |
| GET | `/repayments/my` | EMPLOYEE | Own repayment records |
| GET | `/repayments/employee/:employeeId` | ADMIN | One employee's repayments |
| POST | `/repayments/:id/pay` | ADMIN | Mark repayment as paid |

---

## When is a Repayment Created?

The `Repayment` record is created by `DisbursalsService` at the moment of disbursal — after the money is sent to the employee. It stores the computed breakdown:

```
principalAmount        — the disbursed amount
interestFreeAmount     — portion that accrues no interest
interestBearingAmount  — portion that accrues interest
interestAmount         — computed interest (never changes after creation)
processingFee          — computed processing fee
gstAmount              — computed GST
totalAmount            — what employee owes in total
interestRate           — annual rate % used (frozen from snapshot)
interestDays           — days used in calculation (frozen from snapshot)
dueDate                — recovery date (frozen from snapshot)
status                 — SCHEDULED
```

These values are never recalculated after creation. Even if interest rates change, this repayment stays fixed.

---

## Status Flow

```
SCHEDULED → PAID ✅ (when admin marks it paid after settlement)
          → OVERDUE ❌ (if dueDate passes without payment)
```

---

## Settlement Link

When a settlement is generated, each repayment gets a corresponding `SettlementLineItem` record. A repayment can only be in one settlement (`@unique` on `repaymentId` in `SettlementLineItem`).

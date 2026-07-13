# Module: employer-settlements

Manages the monthly settlement invoices between MobPae and employers.

**Controller:** `src/employer-settlements/employer-settlements.controller.ts`  
**Service:** `src/employer-settlements/employer-settlements.service.ts`

---

## Concept

At the end of each payroll cycle, MobPae generates one settlement invoice per employer. The settlement lists every employee whose advance is due for recovery that month. The employer pays MobPae the total amount; MobPae marks repayments as paid.

---

## Endpoints

| Method | Path | Role | Description |
|---|---|---|---|
| GET | `/employer-settlements` | ADMIN | List all settlements (paginated) |
| GET | `/employer-settlements/employer` | EMPLOYER | Get own settlements |
| GET | `/employer-settlements/employer/summary` | EMPLOYER | Settlement summary stats |
| POST | `/employer-settlements/check-risk/:employerId` | ADMIN | Check employer risk status |
| GET | `/employer-settlements/:id` | ADMIN | Get one settlement with line items |
| POST | `/employer-settlements/:id/send-report` | ADMIN, EMPLOYER | Email settlement report to employer |
| POST | `/employer-settlements/:id/mark-paid` | ADMIN | Mark settlement as paid |

---

## Settlement Structure

```
EmployerSettlement (one per employer per cycle)
  └─ principalAmount    — sum of all principal amounts in cycle
  └─ interestAmount     — sum of all interest amounts
  └─ processingFeeAmount
  └─ gstAmount
  └─ totalAmount        — total MobPae is owed
  └─ outstandingAmount  — decremented as payments arrive
  └─ employeeCount      — number of employees in this settlement
  └─ status             — DRAFT | GENERATED | PARTIALLY_PAID | PAID | OVERDUE | CANCELLED
  └─ lineItems          — one SettlementLineItem per repayment
  └─ payments           — one or more SettlementPayment records

SettlementLineItem (one per employee/advance in the settlement)
  └─ employeeCode, employeeName, loanApplicationNumber  — frozen snapshots
  └─ principalAmount, interestAmount, processingFee, gstAmount, totalDeductionAmount
  └─ repaymentId        — FK to Repayment (unique: each repayment in max one settlement)

SettlementPayment (when employer pays)
  └─ amount             — how much was paid
  └─ paymentMethod      — NEFT | RTGS | IMPS | UPI | NACH | CHEQUE
  └─ bankReference      — employer's UTR/NEFT reference
  └─ transactionDate
  └─ status             — PENDING | VERIFIED | REJECTED
```

---

## Unique Constraint

`(employerId, cycleDate)` is unique — only one settlement per employer per payroll cycle. Attempting to generate a second settlement for the same cycle will return the existing one.

---

## Settlement Number Format

```
MPS-{COMPANYCODE}-{YYYYMM}-{SEQUENCE}
Example: MPS-LTIM-202607-0001
```

---

## Partial Payment Support

A settlement can receive multiple payments (e.g., employer pays in two tranches). Each payment is recorded as a `SettlementPayment` record. The `outstandingAmount` on the settlement is decremented with each verified payment.

Status transitions:
```
DRAFT → GENERATED → PARTIALLY_PAID → PAID ✅
                  → OVERDUE ❌ (if payment not received by dueDate)
```

---

## Risk Status

`Employer.riskStatus` has three values: `GOOD`, `WARNING`, `BLOCKED`.

Admin can trigger a risk check (`POST /employer-settlements/check-risk/:employerId`) which evaluates overdue settlements and updates the risk status accordingly. Blocked employers may have advances paused.

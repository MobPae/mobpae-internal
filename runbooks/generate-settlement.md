# Runbook: Generate Employer Settlement

How to create and send a monthly settlement invoice to an employer.

**Who:** Admin  
**When:** At the start of each payroll cycle (usually the 1st–5th of the month)

---

## What Is a Settlement?

At the end of each payroll cycle, MobPae generates one invoice per employer. The invoice lists every employee whose advance repayment falls due this month. The employer deducts these amounts from payroll and pays MobPae the total.

---

## Steps

### 1. Identify which employers have active repayments

In admin panel: **Settlements** → check for employers with repayments in `SCHEDULED` status for the current month.

Or via API:
```
GET /repayments?status=SCHEDULED
```

### 2. Generate the settlement

**Via API:**
```json
POST /employer-settlements
{
  "employerId": "uuid",
  "cycleDate": "2024-07-01"
}
```

This creates an `EmployerSettlement` with:
- One `SettlementLineItem` per repayment due this cycle
- `status: GENERATED`
- `totalAmount` = sum of all repayment totals
- `outstandingAmount` = same as `totalAmount` initially
- `dueDate` = employer's payroll date this month

**Unique constraint:** If a settlement already exists for this employer + cycle date, the existing one is returned (idempotent).

### 3. Send the settlement report

Email the settlement PDF to the employer:

In admin panel: open the settlement → click **"Send Report"**.

Or via API:
```
POST /employer-settlements/:id/send-report
```

The employer receives an email with the settlement number, amount breakdown, due date, and payment instructions.

### 4. Follow up on payment

Settlement status will be `GENERATED` until payment is received. Monitor via:
```
GET /employer-settlements?status=GENERATED
```

### 5. Record employer payment

When the employer sends the NEFT/RTGS transfer:

In admin panel: open the settlement → click **"Mark as Paid"** → enter the UTR reference number and amount.

Or via API:
```json
POST /employer-settlements/:id/mark-paid
{
  "amount": 52500,
  "paymentMethod": "NEFT",
  "bankReference": "NEFT20240701123456",
  "transactionDate": "2024-07-28T00:00:00Z"
}
```

This:
- Creates a `SettlementPayment` record
- Decrements `outstandingAmount`
- If fully paid: status → `PAID`, all linked `Repayment` records → `PAID`

---

## Partial Payments

If the employer pays in tranches, call `mark-paid` multiple times. `outstandingAmount` decrements each time. Status moves to `PARTIALLY_PAID` after the first payment, and `PAID` when `outstandingAmount = 0`.

---

## Overdue Settlements

If payment is not received by `dueDate`, the settlement status moves to `OVERDUE`. This may trigger a risk status update on the employer.

Run the risk check:
```
POST /employer-settlements/check-risk/:employerId
```

If employer risk becomes `BLOCKED`, new employee advances from that employer will be paused.

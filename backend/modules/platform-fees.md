# Module: platform-fees

Handles the ₹175 per-advance fee paid by the employee after employer approval.

**Controller:** `src/platform-fees/platform-fees.controller.ts`  
**Service:** `src/platform-fees/platform-fees.service.ts`

---

## What is the Platform Fee?

Every time an employer approves a salary advance, the employee must pay a **₹175 platform fee** to MobPae before the advance enters admin review. This is MobPae's primary revenue source.

- Fee is created when employer approves (`ensureFeeForApplication`)
- Paid via Razorpay (online payment in the employee app)
- Can be waived by admin for specific applications
- If payment fails/expires, admin can waive or employee can retry

---

## Endpoints

| Method | Path | Role | Description |
|---|---|---|---|
| GET | `/platform-fees/config` | EMPLOYEE, ADMIN | Get fee amount + Razorpay key |
| GET | `/platform-fees/my` | EMPLOYEE | Get own fee records |
| POST | `/platform-fees/loan-applications/:loanApplicationId/initiate-payment` | EMPLOYEE | Create Razorpay order for fee |
| POST | `/platform-fees/verify-payment` | EMPLOYEE | Client-side verification (optional; webhook is authoritative) |
| GET | `/platform-fees` | ADMIN | List all fees (paginated, filterable) |
| GET | `/platform-fees/:id` | ADMIN | Get one fee record |
| POST | `/platform-fees/:id/waive` | ADMIN | Waive fee for an application |

---

## Fee Lifecycle

```
LoanApplicationFee created (status: PENDING_PAYMENT)
        │
        ├─→ Employee initiates payment
        │     └─ Razorpay Order created (PaymentOrder record)
        │     └─ LoanApplicationFee.providerOrderId set
        │
        ├─→ Employee pays in Razorpay checkout
        │
        ├─→ Razorpay fires payment.captured webhook
        │     └─ LoanApplicationFee status → PAID
        │     └─ LoanApplication status → READY_FOR_DISBURSAL
        │
        ├─→ [Alt] Payment fails
        │     └─ LoanApplicationFee status → FAILED
        │     └─ Employee can retry
        │
        └─→ [Alt] Admin waives
              └─ LoanApplicationFee status → WAIVED
              └─ LoanApplication status → READY_FOR_DISBURSAL
```

---

## Fee Configuration

The platform fee amount (currently ₹175) is stored in the `Settings` table under the key `PLATFORM_FEE_CONFIG`, structured as JSON:

```json
{
  "amount": 175,
  "currency": "INR"
}
```

To change the fee amount: update this setting via the admin panel. All new applications after the change will use the new amount; existing fee records are unaffected.

---

## Key Service Methods

### `getConfig()`
Returns fee amount in INR, amount in paise (for Razorpay), currency, and the Razorpay `keyId`. Used by the employee app before showing the payment UI.

### `ensureFeeForApplication(client, loanApplication)`
Called by `LoanApplicationsService` when employer approves. Creates a `LoanApplicationFee` record if one doesn't already exist. Idempotent — safe to call multiple times.

### `initiatePayment(loanApplicationId, userId)`
1. Validates the fee exists and is `PENDING_PAYMENT`
2. Creates a Razorpay Order via `RazorpayService`
3. Creates a `PaymentOrder` record (purpose: `PLATFORM_FEE`)
4. Returns `{ orderId, amount, currency, keyId }` to the employee app

### `handleWebhookPayment(payload)`
Called by `WebhooksController` when Razorpay fires `payment.captured`. Updates fee status to `PAID` and triggers the application status transition to `READY_FOR_DISBURSAL`.

### `waive(feeId, dto, adminUserId)`
Admin-only. Sets fee status to `WAIVED`, records who waived it and when, triggers application status transition to `READY_FOR_DISBURSAL`.

---

## Database Model: LoanApplicationFee

```
id                String   — UUID
loanApplicationId String   — FK to LoanApplication (unique: one fee per application)
employeeId        String   — FK to Employee
employerId        String   — FK to Employer
feeType           Enum     — PLATFORM_FEE (only type currently)
amount            Decimal  — e.g. 175.00
currency          String   — INR
status            Enum     — PENDING_PAYMENT | PAID | FAILED | EXPIRED | REFUNDED | WAIVED
providerOrderId   String?  — Razorpay order ID
providerPaymentId String?  — Razorpay payment ID (after capture)
providerSignature String?  — Razorpay signature (for verification)
paidAt            DateTime?
waivedAt          DateTime?
waivedBy          String?  — userId of admin who waived
```

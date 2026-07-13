# Runbook: Handle Failed Razorpay Payment

What to do when an employee's ₹175 platform fee payment fails or gets stuck.

**Who:** Admin or Employee  
**When:** Employee pays but status doesn't update, or payment is declined

---

## Symptoms

- Employee paid via Razorpay but application is still in `EMPLOYER_APPROVED`
- Employee reports "payment failed" in the app
- `LoanApplicationFee.status` is `FAILED` or `EXPIRED`

---

## Scenario 1: Payment failed at checkout

The employee's card/UPI was declined. Razorpay shows an error.

**Resolution:**
1. Employee can retry — the app will reuse the same Razorpay order if it hasn't expired (15 min window), or create a new one automatically
2. No admin action needed unless the employee is stuck

---

## Scenario 2: Payment succeeded but webhook didn't fire

The employee paid successfully (Razorpay shows `Captured`) but the application status hasn't moved.

**Check the webhook:**
1. In Razorpay Dashboard → Webhooks → check recent events for `payment.captured`
2. If the webhook failed (4xx/5xx response from backend), retry it from the dashboard

**Check backend logs:**
```bash
# Look for webhook errors
grep "webhook" logs/backend.log | tail -50
```

**Manual fix via API (if webhook keeps failing):**
The `POST /platform-fees/verify-payment` endpoint can also be called manually with the payment details from the Razorpay dashboard:
```json
POST /platform-fees/verify-payment
{
  "razorpayOrderId": "order_xxxx",
  "razorpayPaymentId": "pay_xxxx",
  "razorpaySignature": "hmac_signature"
}
```

> Get the signature from Razorpay dashboard → Payment → Details. If signature isn't available, use the waive option below.

---

## Scenario 3: Order expired (employee waited > 15 min)

Razorpay orders expire after 15 minutes. The `LoanApplicationFee` moves to `EXPIRED`.

**Resolution:**
Employee clicks "Pay ₹175" again → `POST /platform-fees/loan-applications/:id/initiate-payment` creates a fresh Razorpay order → employee completes payment normally.

---

## Scenario 4: Admin decides to waive the fee

For special cases (partner employee, promotional period, payment genuinely stuck):

In admin panel: open the fee record → click **"Waive Fee"**.

Or via API:
```json
POST /platform-fees/:feeId/waive
{ "reason": "Payment confirmed via alternate channel" }
```

This:
- Sets `LoanApplicationFee.status = WAIVED`
- Moves application to `READY_FOR_DISBURSAL` (same effect as successful payment)
- Records the admin who waived it and the reason

---

## Checking Fee Status

```
GET /platform-fees/:id        (admin)
GET /platform-fees/my         (employee — their own fees)
```

Fee statuses: `PENDING_PAYMENT` → `PAID` / `FAILED` / `EXPIRED` / `WAIVED` / `REFUNDED`

---

## Refunds

If an advance is cancelled after payment, the fee may need to be refunded. Currently handled manually via Razorpay dashboard. Update the fee record status to `REFUNDED` via direct database update or a future admin endpoint.

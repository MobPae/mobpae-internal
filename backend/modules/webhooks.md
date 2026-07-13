# Module: webhooks

Handles incoming webhooks from Razorpay.

**Controller:** `src/webhooks/webhooks.controller.ts`

---

## Endpoint

| Method | Path | Auth | Description |
|---|---|---|---|
| POST | `/webhooks/razorpay` | Public (signature verified) | Razorpay event receiver |

---

## Security

Razorpay webhooks are verified using HMAC-SHA256:

```
expectedSignature = HMAC_SHA256(rawBody, RAZORPAY_WEBHOOK_SECRET)
```

The raw request body (not parsed JSON) is required for accurate signature computation. This is why `main.ts` enables raw body parsing for `/webhooks/*` routes.

If the signature doesn't match, the webhook is rejected with 400.

---

## Events Handled

| Event | Action |
|---|---|
| `payment.captured` | Fee paid successfully → update LoanApplicationFee to PAID → LoanApplication to READY_FOR_DISBURSAL |
| `payment.failed` | Fee payment failed → update LoanApplicationFee to FAILED |

---

## Payment Purpose Routing

A single webhook endpoint handles both membership payments and platform fee payments. The handler reads `PaymentOrder.purpose` to route to the correct service:

```
purpose = PLATFORM_FEE → PlatformFeesService.handleWebhookPayment()
purpose = MEMBERSHIP   → MembershipService.handleWebhookPayment()
```

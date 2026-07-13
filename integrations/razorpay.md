# Integration: Razorpay

Used for the ₹175 platform fee payment. Employer pays after approving an advance; the system then clears the application for admin disbursal.

---

## What Razorpay Is Used For

Currently: **Platform Fee collection only** (₹175 per approved advance, paid by employee after employer approval).

Future: May extend to Razorpay Payouts for employee disbursals.

---

## Flow: Platform Fee Payment

```
1. Employer approves application
   → Backend creates LoanApplicationFee record (status: PENDING_PAYMENT)

2. Employee opens fee payment screen
   → App calls: POST /platform-fees/loan-applications/:id/initiate-payment
   → Backend calls Razorpay Orders API to create order
   → Returns: { orderId, amount (paise), key }

3. Employee pays via Razorpay checkout
   → Razorpay JS SDK opens checkout modal
   → Employee selects payment method (UPI / card / netbanking)
   → On success, Razorpay returns: { razorpayOrderId, razorpayPaymentId, razorpaySignature }

4. App verifies payment
   → POST /platform-fees/verify-payment with the three values above
   → Backend verifies HMAC-SHA256 signature
   → If valid: LoanApplicationFee status → PAID
   → LoanApplication status → READY_FOR_DISBURSAL
   → Notification sent to admin

5. (Async) Razorpay webhook fires
   → POST /webhooks/razorpay
   → Backend verifies webhook signature (X-Razorpay-Signature header)
   → Handles: payment.captured event
   → Same effect as step 4 but triggered server-side for reliability
```

---

## Signature Verification

Both payment verification and webhook use HMAC-SHA256.

**Payment verify:**
```
payload = orderId + "|" + paymentId
expected = HMAC-SHA256(payload, RAZORPAY_KEY_SECRET)
compare with razorpaySignature from client
```

**Webhook verify:**
```
expected = HMAC-SHA256(rawBody, RAZORPAY_WEBHOOK_SECRET)
compare with X-Razorpay-Signature header
```

The backend requires raw (unparsed) request body for webhook signature verification. NestJS is configured to preserve raw body via `rawBody: true` in `main.ts`.

---

## Backend Module

**Module:** `src/platform-fees/`  
**Razorpay client:** `src/razorpay/razorpay.service.ts`  
**Webhook handler:** `src/webhooks/webhooks.controller.ts`

The `RazorpayService` wraps the `razorpay` npm package:
```typescript
const razorpay = new Razorpay({
  key_id: process.env.RAZORPAY_KEY_ID,
  key_secret: process.env.RAZORPAY_KEY_SECRET,
});
```

---

## Employee App Integration

The Razorpay checkout JS SDK is loaded in `index.html`:
```html
<script src="https://checkout.razorpay.com/v1/checkout.js"></script>
```

Usage in app:
```javascript
const options = {
  key: razorpayKeyId,
  amount: 17500,  // ₹175 in paise
  currency: 'INR',
  name: 'MobPae',
  description: 'Salary Advance Platform Fee',
  order_id: orderId,
  handler: function(response) {
    // response.razorpay_order_id
    // response.razorpay_payment_id
    // response.razorpay_signature
    verifyPayment(response);
  }
};
const rzp = new window.Razorpay(options);
rzp.open();
```

---

## Environment Variables

```env
RAZORPAY_KEY_ID=rzp_live_xxxx
RAZORPAY_KEY_SECRET=xxxx
RAZORPAY_WEBHOOK_SECRET=xxxx
```

For testing: use `rzp_test_xxxx` key pair and test card numbers from Razorpay docs.

---

## Order Lifecycle

| Event | What happens |
|---|---|
| Order created | `LoanApplicationFee.providerOrderId` set |
| Payment captured | Fee status → PAID; application → READY_FOR_DISBURSAL |
| Payment failed | Fee status → FAILED; employee can retry |
| Order expired (15 min) | Fee status → EXPIRED; new order created on next attempt |
| Webhook received | Idempotent — checks if already processed before acting |

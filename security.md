# Security

Security model, controls, and threat mitigations across MobPae.

---

## Authentication

**JWT Tokens:**
- Access token: 15-minute lifetime, signed with `JWT_SECRET`
- Refresh token: 7-day lifetime, stored hashed in `UserSession` table
- Rotation on refresh — old session is invalidated when a new token pair is issued

**Password hashing:** bcrypt with 10 salt rounds.

**Password reset:** Time-limited token (1 hour), single-use, stored hashed in `PasswordResetToken`.

---

## Role-Based Access Control (RBAC)

Three roles, strictly separated. There is no cross-role access.

| Role | Can access |
|---|---|
| `ADMIN` | Everything. All endpoints. |
| `EMPLOYER` | Only own company's employees, applications, settlements |
| `EMPLOYEE` | Only own profile, own applications, own repayment |

Guards enforced at controller level:
- `JwtAuthGuard` — validates access token on every protected request
- `RolesGuard` — checks `@Roles()` decorator against `req.user.role`

**Key rule: employers cannot access other employers' data.** All employer queries are scoped by `employerId` derived from the JWT, never from a request parameter.

---

## What Each Role Cannot Do

**EMPLOYER cannot:**
- See employees of other employers
- Set their own loan limits (admin-only)
- Verify KYC or bank accounts
- Create or modify loan product config
- Access admin panel endpoints

**EMPLOYEE cannot:**
- See other employees' data (not even peers' real names — peer activity is anonymised)
- Approve or reject their own application
- Modify their loan limit
- Access employer or admin endpoints

**ADMIN cannot:**
- (No restrictions on the platform, but should not share credentials)

---

## File Access Control

All files in R2 are private (no public URLs).

`GET /files/signed-url?key=...` enforces ownership:
- Employee: can only get signed URLs for their own files
- Employer: can only get signed URLs for their own employees' files
- Admin: can get any signed URL

Signed URLs expire in 15 minutes.

---

## Webhook Security

Razorpay webhooks are verified via HMAC-SHA256:

```
expected = HMAC-SHA256(rawBody, RAZORPAY_WEBHOOK_SECRET)
verify: expected === X-Razorpay-Signature header
```

Requests that fail signature verification return `401` immediately. NestJS is configured with `rawBody: true` to preserve the raw request body for this check.

---

## Payment Verification

Client-side Razorpay payments are also verified server-side before any status is updated:

```
payload = orderId + "|" + paymentId
expected = HMAC-SHA256(payload, RAZORPAY_KEY_SECRET)
verify: expected === razorpaySignature from client
```

This prevents a client from claiming a payment was successful without actual Razorpay confirmation.

---

## CORS

CORS is restricted to known frontend origins. Configured via `CORS_ORIGINS` environment variable:

```
CORS_ORIGINS=https://admin.mobpae.com,https://employer.mobpae.com,https://app.mobpae.com
```

In production, wildcard origins (`*`) must never be used.

---

## Security Headers

`helmet` middleware is applied globally in `main.ts`. This sets:
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Strict-Transport-Security` (HSTS)
- `X-XSS-Protection`
- `Referrer-Policy`
- Content Security Policy

---

## Audit Logging

Every admin action is logged to the `AuditLog` table:
- `userId` — who performed the action
- `action` — what was done (e.g., `DISBURSAL_CREATED`, `KYC_VERIFIED`)
- `entityType` + `entityId` — what was affected
- `meta` — relevant details (amounts, reasons)
- `createdAt` — when

This provides a full audit trail for compliance and debugging.

---

## Sensitive Data Handling

- Bank account numbers are stored in plaintext (no encryption at column level currently). **Future:** encrypt `accountNumber` and `ifscCode` at rest.
- KYC document contents are stored in R2 (private bucket) — never in the database
- Passwords are bcrypt-hashed — never stored in plaintext
- JWT secrets and API keys are environment variables — never in code

---

## Known Gaps (To Address)

- Bank account numbers not encrypted at rest
- No rate limiting on auth endpoints (brute force risk)
- No MFA for admin accounts
- Refresh token not invalidated on password change (should be)
- No IP allowlisting for admin panel

---

## Responsible Disclosure

Security issues should be reported to: `security@mobpae.com`  
See also: `compliance/vulnerability-disclosure.md` (if applicable)

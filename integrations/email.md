# Integration: Email / Transactional Notifications

MobPae sends transactional emails for account lifecycle and loan workflow events.

> **Provider: Brevo (formerly Sendinblue)**
> SMTP relay via Nodemailer. Host: `smtp-relay.brevo.com`, Port: `587`.

---

## Emails Sent

### Account / Onboarding

| Trigger | Recipient | Content |
|---|---|---|
| Employee created by employer | Employee | Activation link to set password |
| Employer account created by admin | Employer | Activation link to set password |
| Password reset requested | User | Password reset link (1-hour expiry) |

### Loan Application Workflow

| Trigger | Recipient | Content |
|---|---|---|
| Employee submits advance request | Employer | New request pending approval (with amount and purpose) |
| Employer approves request | Employee | Request approved — fee payment instructions |
| Employer rejects request | Employee | Request rejected with reason |
| Fee payment confirmed | Admin | New application ready for disbursal |
| Admin approves application | Employee | Admin approved — advance disbursing soon |
| Admin rejects application | Employee | Rejected with reason |
| Advance disbursed | Employee | Money sent to bank account |
| Settlement report | Employer | Monthly settlement invoice details |

---

## Notification Model

All notifications are also stored in the `Notification` table in the database, so users can view them in-app even if email is not delivered.

```prisma
model Notification {
  userId    String
  type      String   // APPLICATION_SUBMITTED, FEE_PAID, ADVANCE_DISBURSED, etc.
  title     String
  message   String
  isRead    Boolean  @default(false)
}
```

The in-app notification bell reads from this table. Email is a secondary channel.

---

## Activation Email Flow

When an employee is created, the backend:
1. Creates the `Employee` record with `appActivated: false`
2. Creates the `User` record with a temporary password or null
3. Generates a `PasswordResetToken` with a 72-hour expiry
4. Sends email with link: `{EMPLOYEE_APP_URL}/reset-password?token={selector}:{token}`
5. Employee clicks link → sets their password → `appActivated: true`

Same flow for employer onboarding, pointing to `{EMPLOYER_APP_URL}`.

---

## Environment Variables

```env
# Brevo SMTP relay
SMTP_HOST=smtp-relay.brevo.com
SMTP_PORT=587
SMTP_USER=<brevo-login-email>
SMTP_PASS=<brevo-smtp-key>
SMTP_SECURE=false
MAIL_FROM=support@mobpae.com
MAIL_FROM_NAME=MobPae

# App URLs (used in email links)
EMPLOYEE_APP_URL=https://app.mobpae.com
EMPLOYER_APP_URL=https://employer.mobpae.com
FRONTEND_URL=https://app.mobpae.com   # used for password reset links
```

**Finding your Brevo SMTP key:** Brevo Dashboard → Senders & IPs → SMTP → Generate a new SMTP key. The `SMTP_USER` is your Brevo account email; `SMTP_PASS` is the generated key (not your login password).

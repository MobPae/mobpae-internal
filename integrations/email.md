# Integration: Email / Transactional Notifications

MobPae sends transactional emails for account lifecycle and loan workflow events.

> **Note:** The email provider and template engine are configured in the backend's notification/email module. Check `src/notifications/` or the mail service for the active provider (e.g., SendGrid, Nodemailer, AWS SES).

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
MAIL_HOST=smtp.sendgrid.net
MAIL_PORT=587
MAIL_USER=apikey
MAIL_PASS=SG.xxxx
MAIL_FROM=noreply@mobpae.com
EMPLOYEE_APP_URL=https://app.mobpae.com
EMPLOYER_APP_URL=https://employer.mobpae.com
FRONTEND_URL=https://app.mobpae.com   # used for password reset links
```

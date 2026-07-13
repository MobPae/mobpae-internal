# Module: notifications

In-app notification system for all three user types.

**Controller:** `src/notifications/notifications.controller.ts`

---

## Endpoints

| Method | Path | Role | Description |
|---|---|---|---|
| POST | `/notifications` | ADMIN | Send notification to a user |
| GET | `/notifications` | ADMIN | List all notifications |
| GET | `/notifications/me` | ANY | Get own notifications |
| GET | `/notifications/me/count` | ANY | Get unread count |
| GET | `/notifications/user/:userId` | ADMIN | Get one user's notifications |
| POST | `/notifications/me/read-all` | ANY | Mark all as read |
| POST | `/notifications/:id/read` | ANY | Mark one as read |

---

## Notification Types

| Type | Description |
|---|---|
| `SYSTEM` | In-app notifications (default) |
| `EMAIL` | Email notifications (sent via EmailService) |
| `SMS` | SMS (not implemented yet) |

---

## When Notifications Are Sent

| Event | Recipients |
|---|---|
| Application submitted | Employee (confirmation), Employer (new request) |
| Employer approved | Employee (proceed to pay ₹175) |
| Employer rejected | Employee (reason) |
| Platform fee paid | Employee (under review) |
| Admin approved | Employee |
| Admin rejected | Employee (reason) |
| Disbursal success | Employee (money sent) |
| Repayment marked paid | Employee (cleared) |

---

## Unread Count

The bell icon in the employee app shows a badge with the unread count. The count is fetched from `GET /notifications/me/count` which returns `{ count: number }`.

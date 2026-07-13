# Module: auth

Handles login, token management, and password operations.

**Controller:** `src/auth/auth.controller.ts`

---

## Endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| POST | `/auth/login` | Public | Login with email + password |
| POST | `/auth/refresh` | Public | Get new access token using refresh token |
| POST | `/auth/forgot-password` | Public | Request password reset email |
| POST | `/auth/reset-password` | Public | Reset password via token from email |
| POST | `/auth/logout` | Any | Invalidate current session |
| POST | `/auth/change-password` | Any | Change password (requires current password) |
| GET | `/auth/me` | Any | Get current user profile |

---

## Token Strategy

| Token | Lifetime | Storage |
|---|---|---|
| Access Token (JWT) | 15 minutes | Client memory / localStorage |
| Refresh Token | 7 days | `UserSession` table in DB |

**Access token payload:**
```json
{ "sub": "userId", "role": "EMPLOYEE | EMPLOYER | ADMIN", "iat": ..., "exp": ... }
```

**Refresh flow:**
1. Client sends `POST /auth/refresh` with `{ refreshToken }`
2. Backend looks up `UserSession` by hashed token
3. Validates session is active and not expired
4. Issues new access + refresh token pair
5. Old session is replaced (rotation)

---

## Password Reset Flow

1. Employee/employer submits `POST /auth/forgot-password` with their email
2. Backend creates `PasswordResetToken` (expires in 1 hour, tokenSelector is unique)
3. Email sent with link: `{FRONTEND_URL}/reset-password?token={selector}:{token}`
4. User clicks link → `POST /auth/reset-password` with token + new password
5. Token marked as used; user can log in with new password

---

## Guards

```
JwtAuthGuard  — Validates access token; populates req.user
RolesGuard    — Checks req.user.role against @Roles() decorator
```

Public endpoints are decorated with `@Public()` to skip `JwtAuthGuard`.

---

## User Roles

| Role | Who | Portal |
|---|---|---|
| `ADMIN` | MobPae internal operations team | `mobpae-admin` |
| `EMPLOYER` | HR/finance manager at employer company | `mobpae-employer` |
| `EMPLOYEE` | Salaried employee | `mobpae-app` |

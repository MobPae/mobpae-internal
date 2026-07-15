# Module: Employer Members

Handles multi-user access to the employer portal. Each employer can have multiple users (team members) with role-based permissions.

---

## Models

### EmployerMember
Represents a user's membership in an employer organisation.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `employerId` | uuid | FK → Employer |
| `userId` | uuid | FK → User |
| `role` | EmployerRole | OWNER / ADMIN / HR / FINANCE / VIEWER |
| `status` | EmployerMemberStatus | ACTIVE / SUSPENDED / REMOVED |
| `officeCode` | string? | Optional office/branch code |
| `joinedAt` | DateTime | |
| `removedAt` | DateTime? | Set when removed |

### EmployerInvite
One-time invite sent to an email address. Token is hashed in DB; raw token is emailed.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `employerId` | uuid | FK → Employer |
| `email` | string | Invitee's email |
| `role` | EmployerRole | Role to assign on acceptance |
| `tokenHash` | string | SHA-256 of raw token |
| `status` | InviteStatus | PENDING / ACCEPTED / EXPIRED / REVOKED |
| `expiresAt` | DateTime | 72 hours from send |
| `resendCount` | int | Max 3 resends |
| `lastSentAt` | DateTime | |
| `invitedByUserId` | uuid | FK → User |
| `acceptedByUserId` | uuid? | FK → User — set on acceptance |
| `acceptedAt` | DateTime? | |
| `revokedAt` | DateTime? | |

---

## Roles & Permissions

```
EmployerRole hierarchy (highest → lowest):
  OWNER > ADMIN > HR > FINANCE > VIEWER
```

### Role Capabilities

| Permission | OWNER | ADMIN | HR | FINANCE | VIEWER |
|---|---|---|---|---|---|
| View members/invites | ✅ | ✅ | ✅ | ✅ | ✅ |
| Invite members | ✅ | ✅ | ❌ | ❌ | ❌ |
| Manage members (role/suspend/remove) | ✅ | ✅ | ❌ | ❌ | ❌ |
| View employees | ✅ | ✅ | ✅ | ✅ | ✅ |
| Add/edit employees | ✅ | ✅ | ✅ | ❌ | ❌ |
| Approve/reject salary requests | ✅ | ✅ | ✅ | ❌ | ❌ |
| View settlements | ✅ | ✅ | ❌ | ✅ | ✅ |
| View repayments | ✅ | ✅ | ❌ | ✅ | ✅ |
| View org settings | ✅ | ✅ | ❌ | ❌ | ❌ |
| Manage org settings | ✅ | ❌ | ❌ | ❌ | ❌ |

### Role Management Rules
- A user cannot invite someone to a role equal to or above their own
- Only OWNER can modify or remove another OWNER
- Cannot demote/remove the last OWNER of an organisation

---

## PermissionService

`src/auth/permissions.ts` — `PermissionService` is injectable via `AuthModule`.

```typescript
permissionService.canManageRole(actorRole, targetRole): boolean
// Returns true if actorRole outranks targetRole

permissionService.hasPermission(role, permission): boolean
// Checks role against RolePermissions map
```

---

## EmployerPermissionGuard

`src/auth/guards/employer-permission.guard.ts`

Used alongside `@RequirePermission()` decorator on employer-facing endpoints.

```typescript
@UseGuards(JwtAuthGuard, EmployerPermissionGuard)
@RequirePermission(Permission.MEMBER_INVITE)
createInvite(@Req() req) { ... }
```

Any module using this guard must import `AuthModule` (which exports `PermissionService`).

---

## Auth Middleware Integration

`JwtStrategy` populates `req.user` with `employerId` and `employerRole` by looking up the `EmployerMember` row for the authenticated user. This means:
- All employer endpoints use `req.user.employerId` (not `req.user.userId`) for scoping
- `req.user.employerRole` is the user's role in that employer org

---

## API Endpoints

### Public (no auth)

| Method | Path | Description |
|---|---|---|
| `GET` | `/employer-members/invites/preview?token=` | Get invite details (company, role) before accepting |
| `POST` | `/employer-members/invites/accept` | Accept invite, set password, create account |

### Employer-authenticated

| Method | Path | Permission | Description |
|---|---|---|---|
| `GET` | `/employer-members/invites` | MEMBER_VIEW | List pending invites |
| `POST` | `/employer-members/invites` | MEMBER_INVITE | Send a new invite |
| `POST` | `/employer-members/invites/:id/resend` | MEMBER_INVITE | Resend (regenerates token, resets expiry) |
| `DELETE` | `/employer-members/invites/:id` | MEMBER_INVITE | Revoke a pending invite |
| `GET` | `/employer-members` | MEMBER_VIEW | List active + suspended members |
| `PATCH` | `/employer-members/:id/role` | MEMBER_MANAGE | Change a member's role |
| `PATCH` | `/employer-members/:id/suspend` | MEMBER_MANAGE | Suspend a member |
| `DELETE` | `/employer-members/:id` | MEMBER_MANAGE | Remove a member |

### Admin-authenticated

| Method | Path | Description |
|---|---|---|
| `GET` | `/employers/:id/members` | List all members for any employer |
| `GET` | `/employers/:id/invites` | List pending invites for any employer |

---

## Invite Flow

```
OWNER/ADMIN in portal
  → POST /employer-members/invites { email, role }
  → Backend creates EmployerInvite, emails raw token
  → Invitee opens /invite/accept?token=...
  → GET /employer-members/invites/preview?token= (shows company + role)
  → User sets password → POST /employer-members/invites/accept { token, password }
  → Backend: creates/finds User, upserts EmployerMember, marks invite ACCEPTED
  → User redirected to /login
```

Token details:
- 64-char hex raw token (emailed), SHA-256 hash stored in DB
- Expires 72 hours after send
- Max 3 resends (each resend regenerates token + resets expiry)
- Accepting creates a new User account if the email doesn't exist yet

---

## Seed Script

`prisma/seed-employer-members.ts` — creates an OWNER `EmployerMember` row for each existing `Employer`, linking to the employer's original `userId`. Run after any DB reset.

```bash
npx ts-node prisma/seed-employer-members.ts
```

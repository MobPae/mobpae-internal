# Backend Overview

Repository: `mobpae-backend`  
Framework: NestJS 10 | Language: TypeScript | ORM: Prisma | DB: PostgreSQL

---

## Folder Structure

```
mobpae-backend/
├── prisma/
│   ├── schema.prisma          # Database schema (single source of truth)
│   ├── migrations/            # Auto-generated SQL migration files
│   └── seed.ts                # Demo data seeder
│
├── src/
│   ├── app.module.ts          # Root module — imports all feature modules
│   ├── main.ts                # Bootstrap: CORS, raw body (Razorpay), global pipes
│   │
│   ├── common/
│   │   ├── constants/         # Shared constants
│   │   ├── dto/               # Shared DTOs (pagination)
│   │   ├── enums/             # Shared enums (mirrors Prisma enums)
│   │   ├── interceptors/      # Global interceptors
│   │   └── utils/             # Pagination util, date helpers
│   │
│   ├── auth/                  # JWT auth, login, refresh, password reset
│   ├── users/                 # User model (thin — most data on Employee/Employer)
│   ├── employees/             # Employee CRUD, profile photo
│   ├── employers/             # Employer CRUD, status management
│   ├── kyc-documents/         # KYC upload and admin review
│   ├── bank-accounts/         # Bank account setup and admin verification
│   ├── loan-products/         # Product catalog + versioned config
│   ├── employer-product-configs/ # Per-employer overrides
│   ├── loan-applications/     # Core module: submit, approve, reject, track
│   ├── platform-fees/         # ₹175 fee collection via Razorpay
│   ├── disbursals/            # Admin-initiated payouts
│   ├── repayments/            # Repayment tracking
│   ├── employer-settlements/  # Monthly settlement invoices
│   ├── eligibility/           # Advance limit computation
│   ├── pricing/               # Interest and fee math (pure functions)
│   ├── razorpay/              # Razorpay SDK wrapper
│   ├── webhooks/              # Razorpay webhook handler
│   ├── files/                 # File upload endpoint + signed URL
│   ├── storage/               # Cloudflare R2 service
│   ├── notifications/         # In-app notifications
│   ├── email/                 # Transactional email service
│   ├── audit-logs/            # Action audit trail
│   ├── dashboard/             # Aggregated stats for dashboards
│   ├── reports/               # Financial reports
│   ├── settings/              # Key-value platform settings
│   ├── app-information/       # CMS for T&C, Privacy Policy, FAQs
│   ├── payroll/               # Payroll date utilities
│   ├── salary-requests/       # Legacy alias (maps to loan-applications)
│   ├── salary-limits/         # Legacy alias (maps to loan-limits)
│   ├── sessions/              # Session management
│   ├── business-jobs/         # Background jobs (overdue detection, etc.)
│   ├── pages/                 # Static page content API
│   ├── health/                # Health check endpoint
│   └── prisma/                # PrismaService (shared DB client)
```

---

## Module Anatomy

Each NestJS module follows this pattern:

```
module-name/
├── module-name.module.ts     # Declares providers, imports, exports
├── module-name.controller.ts # HTTP routes + guards + decorators
├── module-name.service.ts    # Business logic
└── dto/
    ├── create-xxx.dto.ts
    ├── update-xxx.dto.ts
    └── xxx-list-query.dto.ts
```

---

## Global Configuration (main.ts)

- **CORS:** Allows requests from all four frontend dev ports (5173–5176) plus the local device IP
- **Raw Body:** Enabled for `/webhooks/*` routes — required for Razorpay HMAC verification
- **Validation Pipe:** Global `ValidationPipe` with `whitelist: true` and `transform: true`
- **Port:** 3000

---

## Auth & Guards

Every protected endpoint has two guards stacked:

```typescript
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('EMPLOYEE', 'EMPLOYER', 'ADMIN')  // specify allowed roles
```

- `JwtAuthGuard`: Verifies the Bearer token; injects `req.user = { userId, role }`
- `RolesGuard`: Compares `req.user.role` against `@Roles(...)` decorator
- Public endpoints (login, refresh, forgot-password, webhook) use `@Public()` decorator to skip guards

---

## Prisma Service

`PrismaService` extends `PrismaClient` and is a global singleton provided to every module. It handles connection on startup and graceful disconnect on shutdown.

---

## Pagination Pattern

All list endpoints follow this pattern:

```typescript
// Query params: ?page=1&limit=20&sortBy=createdAt&sortOrder=desc&search=term
// Response shape:
{
  data: T[],
  meta: {
    page: number,
    limit: number,
    total: number,
    totalPages: number
  }
}
```

Implemented in `src/common/utils/pagination.util.ts`.

---

## Error Handling

Standard NestJS HTTP exceptions are used throughout:

| Exception | HTTP Status | When used |
|---|---|---|
| `NotFoundException` | 404 | Record not found |
| `BadRequestException` | 400 | Invalid input or business rule violation |
| `ForbiddenException` | 403 | Action not allowed for this user |
| `UnauthorizedException` | 401 | Invalid or missing token |
| `ConflictException` | 409 | Duplicate record |
| `ServiceUnavailableException` | 503 | External service (Razorpay) unavailable |

---

## Audit Logging

`AuditLogsService` is injected into any service that performs significant state changes. It records:

```typescript
{
  userId: string,      // Who did it
  action: string,      // e.g. "APPROVE_LOAN_APPLICATION"
  entityType: string,  // e.g. "LoanApplication"
  entityId: string,    // UUID of the affected record
  oldValue: Json,      // State before
  newValue: Json       // State after
}
```

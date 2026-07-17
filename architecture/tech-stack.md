# Tech Stack

---

## Backend

| Technology | Version | Why |
|---|---|---|
| **Node.js** | 20+ | Runtime for NestJS |
| **NestJS** | 10 | Structured, decorator-based framework; excellent module system; built-in DI |
| **TypeScript** | 5 | Type safety across the entire backend |
| **Prisma** | 5 | Type-safe ORM; excellent schema-to-TypeScript generation; migrations |
| **PostgreSQL** | 15+ | Reliable relational DB; strong JSON column support; Prisma native |
| **Passport.js** | — | JWT strategy for authentication |
| **bcrypt** | — | Password hashing |
| **Razorpay Node SDK** | — | Payment orders, payouts, webhook verification |
| **Nodemailer** | — | Transactional email |
| **@aws-sdk/client-s3** | — | Cloudflare R2 is S3-compatible; using AWS SDK to talk to it |
| **class-validator** | — | Request body validation via decorators |
| **class-transformer** | — | DTO serialization / deserialization |

---

## Frontend (All Three Portals)

| Technology | Version | Why |
|---|---|---|
| **React** | 19 | Component model; hooks; great ecosystem |
| **Vite** | 6 | Fast dev server, fast builds |
| **TypeScript** | 5 | Type safety; catches API shape mismatches at compile time |
| **Tailwind CSS** | v4 | Admin + Employer portals; utility-first; custom design tokens via CSS variables |
| **Lucide React** | 0.383 | Icon library; consistent, clean SVG icons |

### Employee App (`mobpae-app`) specific:
- **No external CSS framework** — all styles are inline React styles
- **No React Router** — navigation is a state machine (view string in React state)
- This keeps the bundle small and the app fast on mobile

### Admin & Employer portals specific:
- **Tailwind CSS v4** with shared design tokens in `src/styles/tokens.css`
- **Desktop-first layout** — sidebar navigation, data tables

---

## Infrastructure & Services

| Service | What It Does |
|---|---|
| **Cloudflare R2** | Object storage for KYC documents, profile photos. S3-compatible API. Files are never public — always accessed via signed URLs. |
| **Razorpay** | Payment gateway. Used for: (1) Platform fee collection from employees (₹175 per advance). Future: disbursal payouts to employee bank accounts. |
| **PostgreSQL** | Primary database. All application data: users, loans, repayments, settlements, KYC, notifications. |
| **SMTP / Email** | Transactional emails: password reset, approval/rejection notifications, settlement reports. |

---

## Development Environment

| Tool | Purpose |
|---|---|
| `npx prisma migrate dev` | Apply schema migrations during development |
| `npx prisma generate` | Regenerate Prisma client after schema changes |
| `npx prisma db seed` | Seed demo data for local testing |
| `npx tsc --noEmit` | TypeScript compilation check (0 errors required) |
| `npm run build` | Production build |
| `npm run start:dev` | Dev server with hot reload (NestJS) |
| `npm run dev` | Dev server (Vite frontends) |

---

## Environment Variables (Backend)

Key variables required to run the backend. Full list in `.env.example`.

```
DATABASE_URL          # PostgreSQL connection string
JWT_SECRET            # Access token signing secret
JWT_REFRESH_SECRET    # Refresh token signing secret
JWT_EXPIRES_IN        # e.g. 15m
JWT_REFRESH_EXPIRES_IN # e.g. 7d

RAZORPAY_KEY_ID       # From Razorpay dashboard
RAZORPAY_KEY_SECRET   # From Razorpay dashboard
RAZORPAY_WEBHOOK_SECRET # For webhook signature verification

R2_ACCOUNT_ID         # Cloudflare account ID
R2_ACCESS_KEY_ID      # R2 API token key
R2_SECRET_ACCESS_KEY  # R2 API token secret
R2_BUCKET_NAME        # e.g. mobpae-files
R2_PUBLIC_URL         # Your R2 bucket endpoint

SMTP_HOST             # Email server host
SMTP_PORT             # e.g. 587
SMTP_USER             # Email auth user
SMTP_PASS             # Email auth password
SMTP_FROM             # e.g. noreply@mobpae.com

FRONTEND_URL          # For password reset links
```

# System Overview

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                              │
│                                                                  │
│  ┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
│  │  mobpae-app  │  │ mobpae-employer  │  │  mobpae-admin    │  │
│  │  (Employee)  │  │ (Employer HR)    │  │  (Internal Ops)  │  │
│  │  React+Vite  │  │  React+Vite      │  │  React+Vite      │  │
│  └──────┬───────┘  └────────┬─────────┘  └────────┬─────────┘  │
└─────────┼────────────────────┼────────────────────┼─────────────┘
          │                    │                     │
          └────────────────────┼─────────────────────┘
                               │  HTTPS / REST API
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                       API LAYER                                  │
│                                                                  │
│              mobpae-backend (NestJS)                             │
│              Port 3000                                           │
│                                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │
│  │   Auth   │  │  Loans   │  │Payments  │  │ Settlements  │   │
│  │  Module  │  │  Module  │  │  Module  │  │   Module     │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────┘   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │
│  │Employees │  │Employers │  │  KYC /   │  │  Pricing /   │   │
│  │  Module  │  │  Module  │  │  Files   │  │  Eligibility │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────┘   │
└───────────────────────────────┬─────────────────────────────────┘
                                │
          ┌─────────────────────┼──────────────────────┐
          ▼                     ▼                      ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│   PostgreSQL      │  │  Cloudflare R2   │  │    Razorpay      │
│   (Primary DB)    │  │  (File Storage)  │  │  (Payments)      │
│   via Prisma ORM  │  │  KYC, Selfie,    │  │  Orders &        │
│                   │  │  Profile Photos  │  │  Webhooks        │
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

---

## Applications

### 1. `mobpae-backend`
- **Framework:** NestJS (Node.js)
- **ORM:** Prisma
- **Database:** PostgreSQL
- **Role:** Single API server serving all three frontends
- **Auth:** JWT (access token + refresh token stored in DB)
- **Port:** 3000

### 2. `mobpae-app` (Employee App)
- **Framework:** React 19 + Vite + TypeScript
- **Design:** Mobile-first, dark/light theme, inline styles
- **Routing:** Single-page, view-based (no React Router — internal state machine)
- **Port (dev):** 5175

### 3. `mobpae-admin` (Admin Panel)
- **Framework:** React + Vite + TypeScript
- **Design:** Desktop-first, Tailwind CSS v4 with custom design tokens
- **Port (dev):** 5173

### 4. `mobpae-employer` (Employer Portal)
- **Framework:** React + Vite + TypeScript
- **Design:** Desktop-first, Tailwind CSS v4 with custom design tokens
- **Port (dev):** 5174

### 5. `mobpae-website` (Marketing Site)
- **Framework:** React + Vite + TypeScript
- **Purpose:** Public-facing landing page, employer demo request
- **Port (dev):** 5176

---

## Infrastructure

| Component | Technology | Notes |
|---|---|---|
| Database | PostgreSQL | Managed; connect via `DATABASE_URL` |
| File Storage | Cloudflare R2 | S3-compatible; signed URLs for secure access |
| Payments | Razorpay | Orders for platform fee; Payouts for disbursal (planned) |
| Email | Brevo (SMTP) via Nodemailer | Transactional emails — activation, resets, alerts |
| Hosting | TBD | Backend + frontends deployed separately |

---

## Security Model

- **Authentication:** JWT Bearer tokens (short-lived access + long-lived refresh in DB)
- **Authorization:** Role-based guards on every endpoint (EMPLOYEE / EMPLOYER / ADMIN)
- **File access:** All files in R2 are private; served only via time-limited signed URLs (15-min expiry)
- **Webhooks:** Razorpay webhooks verified with HMAC-SHA256 signature
- **Passwords:** Bcrypt hashed
- **Audit trail:** Every significant action creates an AuditLog record

---

## How the Three Portals Share the Backend

All three frontends call the same NestJS backend. The backend uses role guards to ensure each role can only access its own data:

- `@Roles('EMPLOYEE')` — endpoints only for the employee app
- `@Roles('EMPLOYER')` — endpoints only for the employer portal  
- `@Roles('ADMIN')` — endpoints only for the admin panel
- Some endpoints accept multiple roles (e.g., `@Roles('ADMIN', 'EMPLOYER')`)

The JWT payload contains `{ sub: userId, role: "EMPLOYEE" | "EMPLOYER" | "ADMIN" }`. Role is read from this token, not re-queried from the database on every request.

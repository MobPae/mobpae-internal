# MobPae Internal Knowledge Base

> **Internal use only.** This repository is the single source of truth for MobPae's product, architecture, backend, and frontend. It is intended for the founding team and any new engineers joining the project.

---

## What is MobPae?

MobPae is a fintech platform that enables salaried employees to access their earned wages before payday through employer-backed approvals and NBFC partnerships. The platform bridges the gap between payday and unexpected financial needs without encouraging debt dependency.

**Vision:** Build India's most trusted employer-powered financial wellness ecosystem, enabling responsible financial access while fostering sustainable and inclusive lending.

---

## Quick Navigation

### Product
| Document | Description |
|---|---|
| [Product Overview](./product/overview.md) | Vision, mission, target market, value proposition |
| [User Roles](./product/user-roles.md) | Employee, Employer, Admin — who does what |
| [MVP Features](./product/mvp-features.md) | Every live feature and its current status |
| [User Flows](./product/user-flows.md) | Step-by-step flows for every key journey |

### Architecture
| Document | Description |
|---|---|
| [System Overview](./architecture/system-overview.md) | How all services and apps connect |
| [Tech Stack](./architecture/tech-stack.md) | Every technology used and why |
| [Data Flow](./architecture/data-flow.md) | How data moves through the platform (with diagrams) |

### Backend
| Document | Description |
|---|---|
| [Backend Overview](./backend/overview.md) | NestJS module map, folder structure, conventions |
| [Business Rules](./backend/business-rules.md) | Approval gates, limits, interest logic, fee rules |
| [auth](./backend/modules/auth.md) | JWT auth, roles, password reset |
| [loan-applications](./backend/modules/loan-applications.md) | Core module — full lifecycle |
| [platform-fees](./backend/modules/platform-fees.md) | ₹175 fee, Razorpay, waiver |
| [disbursals](./backend/modules/disbursals.md) | Payout to employee bank account |
| [repayments](./backend/modules/repayments.md) | Repayment tracking and breakdown |
| [employer-settlements](./backend/modules/employer-settlements.md) | Monthly settlement cycle |
| [eligibility](./backend/modules/eligibility.md) | How advance limits are computed |
| [pricing](./backend/modules/pricing.md) | Interest formula and fee math |
| [employees](./backend/modules/employees.md) | Employee onboarding, KYC, selfie |
| [employers](./backend/modules/employers.md) | Employer setup and product config overrides |
| [kyc-documents](./backend/modules/kyc-documents.md) | KYC upload and verification |
| [bank-accounts](./backend/modules/bank-accounts.md) | Bank account setup and verification |
| [notifications](./backend/modules/notifications.md) | In-app notification system |
| [webhooks](./backend/modules/webhooks.md) | Razorpay webhook handling |
| [files-storage](./backend/modules/files-storage.md) | R2 file storage and signed URLs |
| [settings](./backend/modules/settings.md) | System settings and app information |

### Database
| Document | Description |
|---|---|
| [Schema](./database/schema.md) | Every model, every field, every relation explained |
| [Enums](./database/enums.md) | All enums with meaning of each value |
| [Migrations](./database/migrations.md) | Migration history and what each changed |

### API Reference
| Document | Description |
|---|---|
| [Employee App API](./api/employee-app.md) | All endpoints the employee app calls |
| [Employer Portal API](./api/employer-portal.md) | All endpoints the employer portal calls |
| [Admin Panel API](./api/admin-panel.md) | All endpoints the admin panel calls |

### Frontend
| Document | Description |
|---|---|
| [Employee App](./frontend/employee-app.md) | Screens, navigation, state management |
| [Employer Portal](./frontend/employer-portal.md) | Pages, components, data flow |
| [Admin Panel](./frontend/admin-panel.md) | Pages, components, permissions |
| [Shared Patterns](./frontend/shared-patterns.md) | Design tokens, color palette, component patterns |

### Integrations
| Document | Description |
|---|---|
| [Razorpay](./integrations/razorpay.md) | Payment flow, webhook events, order lifecycle |
| [Storage (R2)](./integrations/storage.md) | Cloudflare R2 setup, signed URLs, file types |
| [Email](./integrations/email.md) | Transactional emails and templates |

---

## Repositories

| Repo | Purpose |
|---|---|
| `mobpae-backend` | NestJS API server |
| `mobpae-app` | Employee-facing mobile web app (React + Vite) |
| `mobpae-admin` | Admin panel (React + Vite) |
| `mobpae-employer` | Employer portal (React + Vite) |
| `mobpae-website` | Marketing website (React + Vite) |
| `mobpae-internal` | This repo — internal documentation |

---

## Key Contacts

| Role | Name | Email |
|---|---|---|
| Founder | Jyotish | mob.pae.2026@gmail.com |

---

*Last updated: July 2026*

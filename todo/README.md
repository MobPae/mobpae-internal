# TODO & Future MVP

Tracked work items, deferred features, and upcoming MVP scope. Organised by area.

Status key: 🔜 Planned | 🔧 In Progress | ⏸ Deferred | ✅ Done

---

## Employer Portal (`mobpae-employer`)

| # | Item | Status | Notes |
|---|---|---|---|
| E-01 | Enable Team invite button (remove Beta flag) | ⏸ | Deferred — not in MVP. Backend + accept flow done when ready to enable. |
| E-02 | Accept invite page — mobile responsive | 🔜 | `/invite/accept` page exists but not tested on mobile. |
| E-03 | Employer self-serve onboarding | 🔜 | Allow new employers to sign up from website without admin. Admin reviews + activates. |
| E-04 | Reports page | ✅ | Built: `/reports` with summary cards, monthly trend bar chart (CSS), status breakdown, repayment health, top-10 employee table. Date range filter (1m/3m/6m/all). |
| E-05 | Two-factor auth (MFA) | ⏸ | `mfaEnabled` field exists on User model. No implementation yet. |
| E-06 | Audit log view for employers | 🔜 | Employer should be able to see team actions (who approved what, when). |

---

## Employee App (`mobpae-app`)

| # | Item | Status | Notes |
|---|---|---|---|
| A-01 | Push notifications | ✅ | Done. Capacitor + FCM/APNs integration built. See A-15. |
| A-02 | React Native conversion | 🔜 | Currently React web app running in WebView-like wrapper. Full native rewrite planned. |
| A-03 | Biometric auth | 🔜 | Depends on React Native conversion. |
| A-04 | NACH / UPI Autopay mandate | 🔜 | Set up at onboarding; repayment auto-deducted on payroll date. Removes dependency on employer manually deducting. |
| A-05 | Peer activity feed refinement | 🔧 | Feature exists, needs data accuracy and UX polish. |
| A-06 | Aadhaar OTP / PAN API verification | ⏸ | Deferred to next phase. Currently manual admin KYC. Integrate Digio/Setu/Karza for instant verification. |
| A-07 | Bank account penny drop verification | ⏸ | Deferred to next phase. Instant bank account verification via API instead of manual admin check. |
| A-08 | Email verification on signup | 🔜 | Employee account activation email exists but no "resend verification email" flow. User stuck if email not received. |
| A-09 | Resend OTP / OTP-based login | 🔜 | No OTP login currently. If added, need resend with cooldown (e.g. 60s) and attempt limiting. |
| A-10 | Resend activation email | ✅ | Done. `POST /employees/:id/resend-activation` (ADMIN + EMPLOYER). Generates new temp password, updates hash, resends `employee-created` email. Blocked if `passwordChanged=true`. Employer portal shows "Resend" button per row; admin drawer shows button + "Password set" status field. |
| A-11 | Error state on app load failure | ✅ | Done. `EmployeeApp.tsx` now renders a "Something went wrong" screen with Retry button when `loadState === "error"`. Previously showed infinite skeleton. |
| A-12 | Onboarding setup banner on dashboard | ✅ | Done. `DashboardScreen` shows a persistent blue banner when KYC or bank is incomplete. Banner text mirrors `nextBlocker` state; "Set up →" CTA routes directly to `onboarding-kyc` or `onboarding-bank`. Hidden once employee is eligible. |
| A-13 | Platform fee payment clarity | ✅ | Already implemented. `AdvanceScreen` shows a dedicated gate screen ("One step away") after employer approval — displays approved amount, fee amount, pay CTA via Razorpay, and explains funds move to MobPae review after payment. |
| A-14 | T&C acceptance on first login | 🔜 | Needs a gate screen/modal after forced password change, requiring acceptance of T&C + Privacy Policy before dashboard access. Backend may need `termsAcceptedAt` on `User` model. Legal docs exist (PDF/DOCX). |
| A-15 | Push notifications | ✅ | Done. Architecture: Capacitor wraps existing React/Vite app in native shell. Backend uses `firebase-admin` SDK (`PushNotificationService`). `DeviceToken` Prisma model stores FCM tokens per user (unique, upserted on re-register, auto-pruned on FCM 404). Push triggers: employer approve, employer reject, admin approve, disbursal, KYC verify, KYC reject. Frontend: `pushNotifications.ts` service + `capacitor.config.ts`. Registers on login, cleans up on logout. Browser no-op guard included. **Manual steps still needed:** `npm install firebase-admin` (backend), `npm install @capacitor/core @capacitor/cli @capacitor/push-notifications` (app), `npx cap add ios && npx cap add android`, `npx prisma migrate dev --name add_device_tokens`, set `FIREBASE_PROJECT_ID/CLIENT_EMAIL/PRIVATE_KEY` env vars. |

---

## Admin Panel (`mobpae-admin`)

| # | Item | Status | Notes |
|---|---|---|---|
| D-01 | Financial reports page | ✅ | Done. `DashboardPage` consumes `GET /reports/dashboard` (aggregate platform metrics). `RevenuePage` consumes `GET /reports/revenue` (date-filtered, cross-employer, CSV export). Both wired in sidebar. |
| D-02 | Employer enquiry → employer conversion tracking | 🔜 | Track conversion from leads to active employers. |
| D-03 | Batch disbursal trigger | ⏸ | Not needed — one-by-one disbursal is sufficient for current scale. |
| D-04 | Admin member management (multi-admin) | ⏸ | Only single admin user currently. May need role-based admin access. |

---

## Backend

| # | Item | Status | Notes |
|---|---|---|---|
| B-01 | Razorpay Payouts integration | 🔜 | Automate disbursals via Razorpay Payouts API. Currently manual bank transfer. |
| B-02 | NBFC partner onboarding | 🔜 | Schema ready (`FundingPartner`, `LoanApplication.fundingPartnerId`). No partner yet. |
| B-03 | Credit bureau reporting | ⏸ | Blocked on NBFC licence or partnership. |
| B-04 | Payroll software integrations | 🔜 | Razorpay Payroll, GreytHR, Darwinbox — pull salary data automatically. |
| B-05 | Rate limiting & abuse protection | ✅ | Auth endpoints already throttled. Added `@Throttle` to `invites/preview` (20/15min), `invites/accept` (5/15min), and employer enquiry form (5/hr). |
| B-06 | Background job queue | 🔜 | Email sending, settlement generation, and disbursal triggers are synchronous. Move to a queue (BullMQ/Redis). |
| B-07 | MFA implementation | ⏸ | `mfaEnabled` field exists. TOTP or SMS OTP implementation deferred. |
| B-08 | Refresh token rotation | ✅ | Already fully implemented — `refresh()` uses compare-and-swap (`updateMany` with old hash as condition), race condition guard, and invalidates session on hash mismatch. |
| B-09 | Soft delete pattern | 🔜 | Most deletions are status changes (REMOVED, REVOKED). Review which models need a true soft-delete `deletedAt`. |

---

## Website (`mobpae-website`)

| # | Item | Status | Notes |
|---|---|---|---|
| W-01 | Homepage redesign | 🔧 | In progress on `design/mobpae-website-redesign` branch. New hero, feature sections, stats. |
| W-02 | Product page | 🔜 | Explain salary advance product in detail. |
| W-03 | Employer self-serve enquiry form | ✅ | Form exists on homepage. |
| W-04 | Blog / Content section | ⏸ | For SEO and financial literacy content. Deferred. |
| W-05 | Legal pages inline | 🔜 | Link to Privacy Policy, Terms etc. from footer. Currently only PDF docs exist. |

---

## Internal Docs (`mobpae-internal`)

| # | Item | Status | Notes |
|---|---|---|---|
| I-01 | Runbook: invite flow troubleshooting | 🔜 | What to do when an invite expires, token not received, etc. |
| I-02 | Runbook: reset employer portal access | 🔜 | When the owner loses access. Admin can update user password. |
| I-03 | Architecture diagram (visual) | 🔜 | ASCII/Mermaid diagram of service interactions. |
| I-04 | API changelog | 🔜 | Track breaking changes by date. |
| I-05 | Decision: multi-user employer auth design | 🔜 | Document why EmployerMember was added vs. just using User.role. |

---

## New Financial Products (Long-term MVP)

These are post-MVP features that will use the same infrastructure (loan application flow, product config, settlement):

| Product | Description | Blockers |
|---|---|---|
| Personal Loan (PL) | Unsecured loan for salaried employees | NBFC partnership or NBFC licence |
| Credit Line (CL) | Revolving credit, draw-down as needed | Same as PL |
| Health Insurance EMI | Employer-facilitated insurance via salary deduction | Insurance partner |
| Financial Wellness Suite | Budgeting, savings goals, tax filing | Requires established user base |

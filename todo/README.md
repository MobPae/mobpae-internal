# TODO & Future MVP

Tracked work items, deferred features, and upcoming MVP scope. Organised by area.

Status key: 🔜 Planned | 🔧 In Progress | ⏸ Deferred | ✅ Done

---

## Employer Portal (`mobpae-employer`)

| # | Item | Status | Notes |
|---|---|---|---|
| E-01 | Enable Team invite button (remove Beta flag) | 🔜 | Backend + accept flow done. UI submit is disabled. Wire `POST /employer-members/invites` and enable the button. |
| E-02 | Accept invite page — mobile responsive | 🔜 | `/invite/accept` page exists but not tested on mobile. |
| E-03 | Employer self-serve onboarding | 🔜 | Allow new employers to sign up from website without admin. Admin reviews + activates. |
| E-04 | Reports page | 🔜 | Monthly advance utilisation, repayment rates, per-department breakdown. |
| E-05 | Two-factor auth (MFA) | ⏸ | `mfaEnabled` field exists on User model. No implementation yet. |
| E-06 | Audit log view for employers | 🔜 | Employer should be able to see team actions (who approved what, when). |

---

## Employee App (`mobpae-app`)

| # | Item | Status | Notes |
|---|---|---|---|
| A-01 | Push notifications | 🔜 | In-app only for now. Need FCM/APNs integration. |
| A-02 | React Native conversion | 🔜 | Currently React web app running in WebView-like wrapper. Full native rewrite planned. |
| A-03 | Biometric auth | 🔜 | Depends on React Native conversion. |
| A-04 | NACH / UPI Autopay mandate | 🔜 | Set up at onboarding; repayment auto-deducted on payroll date. Removes dependency on employer manually deducting. |
| A-05 | Peer activity feed refinement | 🔧 | Feature exists, needs data accuracy and UX polish. |
| A-06 | Aadhaar OTP / PAN API verification | 🔜 | Currently manual admin KYC. Integrate Digio/Setu/Karza for instant verification. |
| A-07 | Bank account penny drop verification | 🔜 | Instant bank account verification via API instead of manual admin check. |

---

## Admin Panel (`mobpae-admin`)

| # | Item | Status | Notes |
|---|---|---|---|
| D-01 | Financial reports page | 🔧 | Basic data exists. Need an actual reports UI with date filters, export. |
| D-02 | Employer enquiry → employer conversion tracking | 🔜 | Track conversion from leads to active employers. |
| D-03 | Batch disbursal trigger | 🔜 | Currently one-by-one. Group disbursals by employer and trigger in batch. |
| D-04 | Admin member management (multi-admin) | ⏸ | Only single admin user currently. May need role-based admin access. |

---

## Backend

| # | Item | Status | Notes |
|---|---|---|---|
| B-01 | Razorpay Payouts integration | 🔜 | Automate disbursals via Razorpay Payouts API. Currently manual bank transfer. |
| B-02 | NBFC partner onboarding | 🔜 | Schema ready (`FundingPartner`, `LoanApplication.fundingPartnerId`). No partner yet. |
| B-03 | Credit bureau reporting | ⏸ | Blocked on NBFC licence or partnership. |
| B-04 | Payroll software integrations | 🔜 | Razorpay Payroll, GreytHR, Darwinbox — pull salary data automatically. |
| B-05 | Rate limiting & abuse protection | 🔜 | No throttling on public endpoints yet (invite accept, auth). |
| B-06 | Background job queue | 🔜 | Email sending, settlement generation, and disbursal triggers are synchronous. Move to a queue (BullMQ/Redis). |
| B-07 | MFA implementation | ⏸ | `mfaEnabled` field exists. TOTP or SMS OTP implementation deferred. |
| B-08 | Refresh token rotation | 🔜 | Refresh tokens currently don't rotate on use. Should invalidate old token after each refresh. |
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

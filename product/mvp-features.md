# MVP Features

Status key: ✅ Live | 🔧 Partial / In Progress | 🔜 Planned

---

## Employee App (`mobpae-app`)

| Feature | Status | Notes |
|---|---|---|
| Login / Logout | ✅ | JWT-based, refresh token, session management |
| Forgot / Reset Password | ✅ | Email link with token |
| Change Password | ✅ | In-app from profile |
| Employee Onboarding | ✅ | KYC upload → Bank account → Done screen |
| KYC Upload | ✅ | Aadhaar, PAN, Salary Slip; documents stored in Cloudflare R2 |
| Bank Account Setup | ✅ | Account number, IFSC, UPI; pending admin verification |
| Dashboard | ✅ | Advance limit, salary, active application status |
| Advance Request | ✅ | Amount slider, purpose category, repayment preview |
| Application Status Tracking | ✅ | Real-time status display |
| Platform Fee Payment | ✅ | ₹175 via Razorpay after employer approval |
| Repayment Schedule | ✅ | Amount breakdown, due date, progress bar, auto-deduct label |
| Activity History | ✅ | All past advances (requests + repayments) |
| Notifications | ✅ | In-app bell icon with unread count, mark as read |
| Profile Management | ✅ | Name, photo upload, employer info |
| Help Screen | ✅ | Static help topics, contact info |
| Legal / T&C | ✅ | Links to Privacy Policy, Terms via AppInformation API |
| Dark / Light Theme | ✅ | System-default, toggleable |
| Peer Activity | 🔧 | Shows anonymised company-level advance activity (exists, needs refinement) |
| Push Notifications | 🔜 | Not implemented; in-app only for now |

---

## Employer Portal (`mobpae-employer`)

| Feature | Status | Notes |
|---|---|---|
| Login / Logout | ✅ | |
| Dashboard | ✅ | Summary cards: employees, active advances, pending approvals, settlements |
| Employee Management | ✅ | List, add, edit, activate/deactivate |
| Bulk Employee Upload | ✅ | CSV upload |
| Bulk Activate/Deactivate | ✅ | Multi-select |
| Advance Approval | ✅ | Approve or reject; optional amount override |
| Bulk Approval | ✅ | Approve multiple applications at once |
| Repayments View | ✅ | List of all active and completed repayments |
| Settlements View | ✅ | Monthly settlement list with amounts |
| Settlement Report Download | ✅ | PDF/CSV export |
| Product Config Override | ✅ | Set advance % override for their company |
| Notifications | ✅ | In-app notification bell |
| Settings | ✅ | Company profile, payroll calendar |
| Team Management | 🔧 | Multi-user portal access; invite flow and accept page built; invite button UI gated as Beta |
| Accept Invite Page | ✅ | `/invite/accept?token=...` — public page; shows company + role, sets password, creates account |

---

## Admin Panel (`mobpae-admin`)

| Feature | Status | Notes |
|---|---|---|
| Login / Logout | ✅ | |
| Dashboard | ✅ | Platform-wide metrics |
| Employer Management | ✅ | List, view, approve, reject, suspend |
| Employee Management | ✅ | List, view, verify KYC |
| KYC Review | ✅ | Grouped by employer and employee, verify/reject documents |
| Bank Account Review | ✅ | Verify or reject submitted bank accounts |
| Loan Application Review | ✅ | Final approve/reject after employer |
| Disbursal Management | ✅ | Initiate disbursal, track status |
| Repayments Management | ✅ | View all, mark as paid |
| Settlement Management | ✅ | Generate, view, mark as paid, send report |
| Loan Product Config | ✅ | Edit pricing rules, eligibility rules per product |
| Employer Product Config | ✅ | Per-employer overrides |
| Employer Team Visibility | ✅ | Team members + pending invites visible in employer management drawer |
| Platform Fee Management | ✅ | View fees, waive individual fees |
| Audit Logs | ✅ | Full action history |
| Reports | 🔧 | Basic financial reports; more planned |
| App Information Management | ✅ | Manage T&C, Privacy Policy, FAQs in-app |
| Notifications | ✅ | Send to any user |
| Settings | ✅ | Key-value platform settings |

---

## Backend / Platform

| Feature | Status | Notes |
|---|---|---|
| Multi-role Auth (JWT) | ✅ | EMPLOYEE / EMPLOYER / ADMIN |
| Multi-user Employer Auth | ✅ | EmployerMember model; JWT carries `employerId` + `employerRole`; EmployerPermissionGuard on all employer endpoints |
| Employer Role Hierarchy | ✅ | OWNER > ADMIN > HR > FINANCE > VIEWER; per-role permission map |
| Refresh Token Sessions | ✅ | Stored in DB; invalidated on logout |
| Role-based Access Control | ✅ | Guards on every endpoint |
| Versioned Loan Product Config | ✅ | Immutable once active; version chain |
| Per-employer Product Overrides | ✅ | % or absolute ₹ cap |
| Eligibility Computation | ✅ | Dynamic: salary, %, cooldown, existing advances |
| Pricing / Interest Calculation | ✅ | Simple daily interest, interest-free threshold |
| Snapshot at Submission | ✅ | Rates frozen on LoanApplication at submit time |
| Platform Fee via Razorpay | ✅ | ₹175 Razorpay order, webhook confirmation |
| Admin Fee Waiver | ✅ | Admin can waive fee, skips to READY_FOR_DISBURSAL |
| Disbursal Record | ✅ | Bank snapshot, provider reference, retry count |
| Repayment Record | ✅ | Full breakdown; linked to settlement line items |
| Employer Settlement | ✅ | Monthly cycle, line items, payment tracking |
| File Storage (Cloudflare R2) | ✅ | KYC, profile photo; signed URL access |
| Email Notifications | ✅ | Transactional emails (approval, rejection, etc.) |
| Audit Logging | ✅ | Every significant action logged |
| Payroll Calendar Logic | ✅ | Recovery date computed from employer's payroll day and cutoff day |
| Funding Partner Model | ✅ | Schema ready; self-funded now; NBFC-ready |
| NBFC Integration | 🔜 | Schema and model in place; no partner onboarded yet |
| NACH / Auto-debit | 🔜 | Planned; currently manual salary deduction by employer |
| Push Notifications | 🔜 | Planned |
| Credit Bureau Reporting | 🔜 | Planned when NBFC partners are onboarded |

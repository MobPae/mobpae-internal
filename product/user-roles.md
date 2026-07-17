# User Roles

MobPae has three distinct user roles. Each role maps to a separate interface and a separate set of backend permissions.

---

## 1. Employee

**Who:** A salaried employee at an onboarded employer company.

**Interface:** `mobpae-app` — mobile-first web app, accessed on a phone browser or as a PWA.

**How they join:** The employer uploads them via bulk CSV or adds them individually through the employer portal. The employee then activates their account by setting a password via a link sent to their email.

**What they can do:**

| Action | Notes |
|---|---|
| View their advance limit | Computed from salary, employer config, and admin-set loan limit |
| Submit a salary advance request | Select amount, category, confirm repayment preview |
| Track application status | See real-time status updates as employer / admin acts |
| Pay the ₹175 platform fee | Via Razorpay after employer approval |
| View repayment schedule | Exact amount due, due date, breakdown |
| Upload KYC documents | Aadhaar, PAN, Salary Slip |
| Add/view bank account | Salary account for advance disbursal |
| View activity history | All past advances and repayments |
| Manage profile | Name, photo, change password |
| View notifications | In-app alerts for status changes |

**Constraints:**
- One active advance at a time (configurable)
- Cannot submit a new advance while an existing one is in-flight
- Advance amount capped by eligibility computation

---

## 2. Employer

**Who:** The HR manager or finance manager at a company that has signed up with MobPae.

**Interface:** `mobpae-employer` — web portal, accessed on a desktop browser.

**How they join:** MobPae admin creates the employer account and sends login credentials.

**What they can do:**

| Action | Notes |
|---|---|
| Add employees | Individually or via bulk CSV upload |
| Activate / deactivate employees | Controls whether an employee can use the app |
| Review and approve/reject advance requests | The first approval gate in the flow |
| Bulk approve/reject | Multiple applications at once |
| View repayments | See all active and completed repayments |
| View settlements | Monthly settlement invoices from MobPae |
| Download settlement reports | For payroll reconciliation |
| Configure advance percentage | Override the default advance % (within MobPae's hard ceiling) |
| View employee KYC/bank status | Read-only |

**Key constraint:** Employer can only view and act on their own employees. They cannot see any other employer's data.

---

## 3. Admin (MobPae Internal)

**Who:** MobPae's internal operations team.

**Interface:** `mobpae-admin` — web portal, accessed on a desktop browser.

**What they can do:**

| Action | Notes |
|---|---|
| Manage employers | Approve, reject, suspend employer accounts |
| Manage employees | View, update, verify KYC |
| Final approval/rejection of advances | Second approval gate after employer |
| Initiate disbursals | Send money to employee bank account |
| View and manage repayments | Mark as paid, track overdue |
| Generate employer settlements | Monthly payroll recovery invoices |
| Mark settlements as paid | When employer transfers funds |
| Manage loan products | Configure pricing rules, eligibility rules |
| Manage employer product configs | Per-employer overrides |
| Waive platform fees | Exemption for specific applications |
| View audit logs | Full history of all actions on any record |
| Manage app content | Terms, Privacy Policy, FAQs (via AppInformation) |
| View reports | Financial summaries |
| Manage platform fee config | Change ₹175 amount if needed |

**Admin has full visibility across all employers and employees.**

---

## Role Comparison

| Feature | Employee | Employer | Admin |
|---|---|---|---|
| Submit advance | ✅ | — | — |
| Employer approval | — | ✅ | — |
| Admin approval | — | — | ✅ |
| Initiate disbursal | — | — | ✅ |
| View own data | ✅ | ✅ (company) | ✅ (all) |
| Manage users | — | ✅ (own employees) | ✅ (all) |
| Configure products | — | ✅ (limited) | ✅ (full) |
| Settlement management | — | View only | ✅ full |
| Waive fees | — | — | ✅ |

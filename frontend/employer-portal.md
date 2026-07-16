# Frontend: Employer Portal (mobpae-employer)

**Repo:** `mobpae-employer`  
**Stack:** React 19, TypeScript, Vite, Tailwind CSS v4, React Router v6  
**Target:** Desktop web browser

---

## Architecture

Standard React SPA with file-based routing via React Router. State management is local — no global store. Each page fetches its own data.

```
src/
├── main.tsx
├── pages/              (route-based pages)
│   ├── auth/
│   ├── dashboard/
│   ├── employees/
│   ├── salary-requests/
│   ├── recoveries/
│   ├── repayments/
│   ├── reports/
│   ├── settlements/
│   ├── team/
│   └── settings/
├── components/
│   ├── layout/         (EmployerLayout, NotificationBell)
│   ├── ui/             (shared primitives)
│   └── employees/      (feature components)
├── hooks/
│   ├── useAuth.tsx     (login state)
│   └── useToast.tsx    (toast notifications)
└── services/           (API calls)
```

---

## Pages

| Route | Page | Description |
|---|---|---|
| `/login` | LoginPage | Email/password |
| `/invite/accept` | AcceptInvitePage | Public — validates invite token, sets password, creates account |
| `/forgot-password` | ForgotPasswordPage | Reset request |
| `/reset-password` | ResetPasswordPage | Set new password |
| `/change-password` | ChangePasswordPage | Update current password |
| `/` or `/dashboard` | DashboardPage | Summary stats, pending approvals |
| `/employees` | EmployeesPage | Employee list, add/edit/bulk |
| `/loan-applications` | SalaryRequestsPage | All requests with approve/reject |
| `/repayments` | RepaymentsPage | Repayment schedule per employee |
| `/settlements` | SettlementsPage | Monthly settlement invoices |
| `/team` | TeamPage | Team members + pending invites; role-based actions (Beta) |
| `/reports` | ReportsPage | Advance utilisation, repayment health, monthly trend, employee breakdown |
| `/settings` | SettingsPage | Company profile |
| `/profile` | ProfilePage | User profile |
| `/notifications` | NotificationsPage | In-app notifications |

---

## Layout

`EmployerLayout` is the shell for all authenticated pages. It provides:
- Left sidebar with navigation links
- Top header with company name and notification bell
- Main content area

Sidebar links: Dashboard, Employees, Loan Applications, Repayments, Settlements, Reports, Salary Advance (settings), Team, Profile.

---

## Key Components

### UI Primitives (`src/components/ui/`)

| Component | Purpose |
|---|---|
| `Button` | Styled button with variants (primary/secondary/danger) |
| `Input` | Text input with label and error |
| `Select` | Dropdown select |
| `DataTable` | Sortable/filterable table |
| `Pagination` | Page navigation |
| `Drawer` | Right-side slide-in panel for detail views |
| `ConfirmModal` | Confirmation dialog (destructive actions) |
| `StatusBadge` | Color-coded status chip |
| `MetricCard` | Stats card with number + label |
| `PageHeader` | Page title + breadcrumb |

### Employee Components (`src/components/employees/`)

| Component | Purpose |
|---|---|
| `EmployeeTable` | List employees with search/filter |
| `AddEmployeeDrawer` | Slide-in form to create one employee |
| `EditEmployeeDrawer` | Slide-in form to edit employee |
| `BulkEmployeeForm` | Paste or upload CSV for bulk create |

---

## Team Page

`/team` — `src/pages/team/TeamPage.tsx`

Shows two tabs: **Members** and **Invites**.

- **Members tab**: lists all active/suspended team members with role badge, status badge, and joined date. Action menu (change role / suspend / remove) visible only to OWNER and ADMIN.
- **Invites tab**: lists pending invites with role, expiry, and Resend/Revoke actions (OWNER/ADMIN only).
- **Invite drawer**: OWNER/ADMIN can open an invite form. Currently marked as **Beta** — submit button is disabled with an amber notice until the invite accept flow is fully tested in production.

Permission gating:
```typescript
const canManage = user?.employerRole === "OWNER" || user?.employerRole === "ADMIN";
```

---

## Reports Page

`/reports` — `src/pages/reports/ReportsPage.tsx`

Advance utilisation and repayment overview for the employer. Data is fetched from existing endpoints — no dedicated reports backend endpoint.

**Data sources:**
- `salaryRequestService.getSalaryRequests()` → `GET /loan-applications/employer`
- `repaymentService.getRepayments()` → `GET /repayments/employer`

**Date range filter:** This month / Last 3 months / Last 6 months / All time. Filter applies to all sections except the monthly trend chart (which always shows last 6 months).

**Sections:**
1. **Summary cards** — Total requests, Amount disbursed, Approval rate, Unique employees who used advances
2. **Monthly trend bar chart** — Disbursed advances per month for last 6 months (CSS/div-based, no charting library)
3. **Application status breakdown** — Stacked bar: Active/Disbursed, Repaid, Pending review, Rejected
4. **Repayment health** — Stacked bar: Paid, Scheduled, Overdue; overdue amount alert shown if > 0
5. **Top employees table** — Top 10 by total advance amount: name, code, count, total, last advance date

**Important:** `mobpae-employer` has no charting library (recharts, chart.js etc.). All charts are CSS div bars with percentage-based widths via inline styles.

---

## Accept Invite Page

`/invite/accept?token=...` — `src/pages/auth/AcceptInvitePage.tsx`

Public page (no auth required). Flow:
1. On mount: `GET /employer-members/invites/preview?token=` — shows company name + role if valid
2. User sets a password and submits: `POST /employer-members/invites/accept { token, password }`
3. On success: redirected to `/login` with a success message
4. Error states: invalid token, expired token, password mismatch

Uses `httpClient` (unauthenticated axios instance) directly since the user has no JWT yet.

---

## Auth Flow

`useAuth` hook manages JWT tokens. Tokens stored in `localStorage`.

- Login → store `accessToken` + `refreshToken`
- All API calls attach `Authorization: Bearer <accessToken>`
- On 401 → attempt refresh via `POST /auth/refresh`
- If refresh fails → redirect to `/login`

`ProtectedRoute` wrapper redirects to `/login` if not authenticated.

---

## Salary Request Approval Flow

1. **SalaryRequestsPage** fetches `GET /loan-applications/employer`
2. Pending requests shown with "Approve" / "Reject" buttons
3. Approve: `POST /loan-applications/:id/employer-approve` — no body needed
4. Reject: opens modal, submits `POST /loan-applications/:id/employer-reject` with reason
5. Bulk approve/reject: `POST /loan-applications/bulk-action` with array of IDs

---

## Settlements Flow

1. **SettlementsPage** fetches `GET /employer-settlements/employer`
2. Each settlement shows total amount, outstanding, due date, status
3. Clicking a settlement opens a drawer with line items (per employee)
4. "Send Report" button triggers `POST /employer-settlements/:id/send-report`

---

## Styling

Tailwind CSS v4. Shared design tokens defined in CSS custom properties:
- `--color-primary`, `--color-surface`, `--color-border`, etc.
- Both admin and employer portal share the same token names for consistency.

---

## Key Files

| File | Purpose |
|---|---|
| `src/hooks/useAuth.tsx` | Login state, token management, API auth |
| `src/components/layout/EmployerLayout.tsx` | App shell with sidebar |
| `src/pages/salary-requests/SalaryRequestsPage.tsx` | Core approval workflow |
| `src/pages/settlements/SettlementsPage.tsx` | Settlement management |
| `src/pages/reports/ReportsPage.tsx` | Advance utilisation and repayment reports |
| `src/pages/team/TeamPage.tsx` | Team member + invite management |
| `src/pages/auth/AcceptInvitePage.tsx` | Public invite acceptance flow |

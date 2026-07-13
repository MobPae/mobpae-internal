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
│   ├── settlements/
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
| `/forgot-password` | ForgotPasswordPage | Reset request |
| `/reset-password` | ResetPasswordPage | Set new password |
| `/change-password` | ChangePasswordPage | Update current password |
| `/` or `/dashboard` | DashboardPage | Summary stats, pending approvals |
| `/employees` | EmployeesPage | Employee list, add/edit/bulk |
| `/salary-requests` | SalaryRequestsPage | All requests with approve/reject |
| `/repayments` | RecoveriesPage | Repayment schedule per employee |
| `/settlements` | SettlementsPage | Monthly settlement invoices |
| `/settings` | SettingsPage | Company profile |

---

## Layout

`EmployerLayout` is the shell for all authenticated pages. It provides:
- Left sidebar with navigation links
- Top header with company name and notification bell
- Main content area

Sidebar links: Dashboard, Employees, Salary Requests, Repayments, Settlements, Settings.

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

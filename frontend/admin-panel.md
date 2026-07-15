# Frontend: Admin Panel (mobpae-admin)

**Repo:** `mobpae-admin`  
**Stack:** React 19, TypeScript, Vite, Tailwind CSS v4, React Router v6  
**Target:** Desktop web browser (internal MobPae operations tool)

---

## Architecture

Same pattern as employer portal — file-based routing, local state, no global store.

```
src/
├── main.tsx
├── App.tsx              (route definitions)
├── pages/               (one folder per feature)
│   ├── auth/
│   ├── dashboard/
│   ├── employees/       (KYC, bank verification, employee management)
│   ├── employers/       (employer management)
│   ├── loan-applications/
│   ├── disbursals/
│   ├── repayments/
│   ├── settlements/
│   ├── platform-fees/
│   ├── settings/
│   └── loan-products/
├── components/
│   ├── layout/          (AdminLayout, Sidebar, Header, NotificationBell)
│   ├── ui/              (shared primitives — same as employer)
│   ├── kyc/
│   ├── bank-verification/
│   ├── employers/
│   ├── disbursals/
│   ├── dashboard/
│   └── auth/
└── hooks/
    └── useSignedUrl.ts  (fetch signed URLs for KYC docs, selfies)
```

---

## Pages

| Route | Description |
|---|---|
| `/login` | Admin login |
| `/` | Dashboard — action queue, stats, recent activity |
| `/employers` | Employer list, create, status management |
| `/employees` | All employees with KYC/bank/selfie status |
| `/kyc` | KYC document review (grouped by employee) |
| `/bank-verification` | Bank account verification queue |
| `/loan-applications` | All salary advance requests |
| `/disbursals` | Disbursal management (create + trigger) |
| `/repayments` | All repayment records |
| `/settlements` | All employer settlements |
| `/platform-fees` | Platform fee records, waive |
| `/settings` | Platform config, app information |
| `/loan-products` | Loan product config management |

---

## Layout

`AdminLayout` wraps all authenticated pages. Contains:
- Left sidebar with full navigation
- Top header with admin name and notification bell

---

## Key Feature Components

### KYC Review (`src/components/kyc/`)

- `KycGroupedTable` — lists employees grouped by KYC status
- `KycGroupedDrawer` — shows all documents for one employee; verify/reject each
- `KycDrawer` — single document detail with signed URL image preview
- Uses `useSignedUrl` hook to fetch 15-min signed URLs for each document image

### Bank Verification (`src/components/bank-verification/`)

- `BankGroupedTable` — employees with unverified bank accounts
- `BankVerificationDrawer` — shows bank details, admin can verify or flag

### Employers (`src/components/employers/`)

- `EmployersTable` — paginated list with status, risk status filters
- `EmployerManagementDrawer` — full employer detail; change status, override product config, view team members and pending invites
- `CreateEmployerDrawer` — form to create a new employer

**EmployerManagementDrawer sections:**
1. Company details (code, risk status, member since)
2. Contact information
3. Salary cycle configuration
4. Salary advance override (editable ₹ cap)
5. Recent salary requests
6. **Team members** — lists active/suspended members with role badge, status, joined date
7. **Pending invites** — lists open invites with role badge and expiry (hidden if none exist)
8. Footer actions: Activate / Suspend / Reactivate / Generate Settlement

### Disbursals (`src/components/disbursals/`)

- `DisbursalsTable` — all disbursals with status filter
- `DisbursalDrawer` — create disbursal from an approved application, trigger payout

### Dashboard (`src/components/dashboard/`)

- `ActionQueue` — pending KYC reviews, disbursals, applications needing action
- `FinancialOverview` — total disbursed, outstanding, collected
- `RecentSalaryRequests` — last 10 applications
- `SystemOverview` — employer count, employee count, active advances

---

## Signed URL Pattern for Private Files

KYC documents and selfies are stored in Cloudflare R2 as private objects. The admin panel fetches a signed URL before displaying any image.

```typescript
// src/hooks/useSignedUrl.ts
function useSignedUrl(fileKey: string | null | undefined) {
  const [url, setUrl] = useState<string | null>(null);
  useEffect(() => {
    if (!fileKey) return;
    api.getSignedUrl(fileKey).then(({ url }) => setUrl(url));
  }, [fileKey]);
  return url;
}
```

Usage in components:
```tsx
const photoUrl = useSignedUrl(employee.profilePhotoUrl);
<img src={photoUrl ?? placeholder} />
```

Signed URLs expire in 15 minutes. If an image fails to load, the user can refresh the drawer.

---

## Auth & Protected Routes

Same pattern as employer portal:
- `useAuth` hook manages JWT storage and refresh
- `ProtectedRoute` component guards all authenticated routes
- On 401 → refresh attempt → redirect to `/login` on failure

---

## Loan Product Config Page

`/loan-products` allows admin to:
1. View active config (eligibility rules, pricing rules, operational rules)
2. Create a new config version (old version is deactivated, cannot be edited)

The config is a JSON object stored in `LoanProductConfig.eligibilityRules`, `.pricingRules`, `.operationalRules`. The form presents these as individual fields.

---

## Tailwind Tokens

Shared with employer portal. Both use the same CSS custom property names. Defined in:
- `mobpae-employer/src/styles.css`
- `mobpae-admin/mobpae-admin/src/styles.css` (or equivalent)

Key tokens: `--color-primary`, `--color-surface-1`, `--color-surface-2`, `--color-border`, `--color-text-primary`, `--color-text-muted`, `--color-success`, `--color-warning`, `--color-danger`.

---

## Key Files

| File | Purpose |
|---|---|
| `src/App.tsx` | All route definitions |
| `src/components/layout/AdminLayout.tsx` | App shell |
| `src/components/layout/Sidebar.tsx` | Navigation sidebar |
| `src/components/kyc/KycGroupedDrawer.tsx` | Primary KYC review UI |
| `src/components/disbursals/DisbursalDrawer.tsx` | Disbursal creation + trigger |
| `src/hooks/useSignedUrl.ts` | Private file URL resolution |

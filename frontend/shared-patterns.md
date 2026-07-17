# Frontend Shared Patterns

Design conventions and patterns shared across all MobPae frontends.

---

## Color Palette

### Brand Colors

| Name | Hex | Usage |
|---|---|---|
| Primary Blue | `#2563EB` | CTAs, links, active states |
| Primary Dark | `#1D4ED8` | Hover states |
| Success Green | `#16A34A` | Approved, paid, verified states |
| Warning Amber | `#D97706` | Pending, warning states |
| Danger Red | `#DC2626` | Rejected, error, overdue states |
| Neutral Gray | `#6B7280` | Muted text, borders |

### Semantic CSS Variables (Tailwind Portals)

```css
--color-primary       /* Brand blue */
--color-surface-1     /* Page background */
--color-surface-2     /* Card background */
--color-border        /* Default border */
--color-text-primary  /* Body text */
--color-text-muted    /* Secondary text */
--color-success       /* Green */
--color-warning       /* Amber */
--color-danger        /* Red */
```

Used in both `mobpae-admin` and `mobpae-employer`. Ensures both portals look consistent without a shared component library.

---

## Status Badge Colors

Used consistently across all portals:

| Status | Color |
|---|---|
| SUBMITTED | Blue (info) |
| EMPLOYER_APPROVED | Teal |
| EMPLOYER_REJECTED | Red |
| AWAITING_PLATFORM_FEE_PAYMENT | Amber |
| READY_FOR_DISBURSAL | Purple |
| ADMIN_REJECTED | Red |
| DISBURSED | Blue |
| REPAYMENT_SCHEDULED | Blue |
| REPAID | Green |
| CANCELLED | Gray |
| OVERDUE | Red |
| PENDING | Amber |
| VERIFIED | Green |
| REJECTED | Red |
| ACTIVE | Green |
| INACTIVE | Gray |
| SUSPENDED | Red |

---

## Typography

**Employee App:** System font stack with custom CSS properties. No external font import.  
**Employer/Admin:** Same — system UI fonts via Tailwind.

Font scale:
- Headings: `text-xl` / `text-2xl` (Tailwind) or `--font-size-xl` (app CSS)
- Body: `text-sm` / `text-base`
- Captions / muted: `text-xs`

---

## API Layer Pattern

All three frontends follow the same pattern for API calls:

```typescript
// services/api.ts or similar
const BASE_URL = import.meta.env.VITE_API_URL ?? 'http://localhost:3000';

async function apiFetch<T>(path: string, options: RequestInit = {}): Promise<T> {
  const token = localStorage.getItem('accessToken');
  const res = await fetch(`${BASE_URL}${path}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...options.headers,
    },
  });
  if (res.status === 401) {
    // attempt refresh → retry or redirect
  }
  if (!res.ok) throw new Error(await res.text());
  return res.json();
}
```

No external HTTP client (no axios). Native `fetch` throughout.

---

## Auth Token Storage

| Frontend | Storage |
|---|---|
| Employee App | `localStorage` |
| Employer Portal | `localStorage` |
| Admin Panel | `localStorage` |

Tokens stored as: `accessToken`, `refreshToken`.  
On page load: check `localStorage` → if token exists, validate by calling `GET /auth/me` or equivalent.

---

## Error Handling

- Network errors → shown inline as error text or toast
- 401 → auto-refresh attempt, then redirect to login
- 422/400 validation errors → form field error display
- 500 → generic "Something went wrong" message

---

## Environment Variables

All frontends use Vite env variables prefixed with `VITE_`:

```env
VITE_API_URL=https://api.mobpae.com
VITE_RAZORPAY_KEY_ID=rzp_live_xxxx   # employee app only
```

Accessed as `import.meta.env.VITE_API_URL`.

---

## File Upload Pattern

KYC documents and profile photos are uploaded as `multipart/form-data`:

```typescript
const formData = new FormData();
formData.append('file', file);
formData.append('documentType', 'AADHAR');

await fetch('/kyc-documents', {
  method: 'POST',
  headers: { Authorization: `Bearer ${token}` },
  body: formData,
  // NOTE: do NOT set Content-Type; let browser set multipart boundary
});
```

---

## Pagination Pattern (Tailwind Portals)

Backend returns:
```json
{ "data": [...], "total": 150, "page": 1, "limit": 20 }
```

Frontend tracks `page` in local state. `Pagination` component emits `onPageChange(page)` which triggers a new fetch.

---

## Notification Bell

Both admin and employer portals have a `NotificationBell` component in the header. It:
1. Fetches `GET /notifications/my` on mount
2. Shows a red dot if `unreadCount > 0`
3. Opens a dropdown with the notification list
4. Calls `POST /notifications/mark-all-read` when dropdown is opened

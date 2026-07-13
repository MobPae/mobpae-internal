# Frontend: Employee App (mobpae-app)

**Repo:** `mobpae-app`  
**Stack:** React 19, TypeScript, Vite, custom CSS (no component library)  
**Target:** Mobile browser (PWA-style, fixed 390px width shell)

---

## Architecture

The app is a single-page application with a custom view router. There is no React Router — navigation is managed with a `activeView` state variable in `useEmployeeApp`.

```
main.tsx
└── EmployeeApp.tsx  (root component, view router)
    ├── hooks/useEmployeeApp.ts  (all state + API calls)
    ├── hooks/useTheme.ts        (dark/light mode)
    ├── hooks/useSignedUrl.ts    (signed URL fetching for private files)
    └── screens/                 (one component per screen)
```

---

## Screens

| Screen | View Key | Description |
|---|---|---|
| `LoginScreen` | `home` (unauthenticated) | Email/password login |
| `ForgotPasswordScreen` | `forgot-password` | Request reset email |
| `ResetPasswordScreen` | `reset-password` | Enter new password via token |
| `DashboardScreen` | `home` (authenticated) | Hero card, eligibility, quick actions |
| `AdvanceScreen` | `advance` | Amount selector, repayment preview, apply CTA |
| `ActivityScreen` | `activity` | Tabs: Active requests / Repayments / History |
| `RepaymentScheduleScreen` | `repayment` | Breakdown of scheduled repayment |
| `TrackingScreen` | `tracking` | Track active request through status stages |
| `MembershipScreen` | `membership` | Platform info / tier display |
| `ProfileScreen` | `profile` | User profile, settings links |
| `OnboardingKycScreen` | `onboarding-kyc` | Upload KYC documents |
| `OnboardingBankScreen` | `onboarding-bank` | Add bank account |
| `OnboardingDoneScreen` | `onboarding-done` | Onboarding completion screen |
| `NotificationsScreen` | `notifications` | Notification list |
| `HelpScreen` | `help` | FAQ / help topics |
| `LegalScreen` | `legal` | Terms, Privacy Policy links |
| `ChangePasswordScreen` | `change-password` | Update password |

---

## Navigation Pattern

No React Router. All navigation goes through `app.setActiveView(viewKey)`.

The `EmployeeApp.tsx` component is a large conditional render block:
```tsx
if (!app.isLoggedIn) → render auth screens
if (app.activeView === 'advance') → render AdvanceScreen
if (app.activeView === 'activity') → render ActivityScreen
// etc.
```

Tab bar (bottom navigation) is rendered inside `AppShell` and always visible when authenticated.

---

## Central State Hook: `useEmployeeApp`

Located at `src/hooks/useEmployeeApp.ts`. This is the single source of truth for the entire app.

**Key state:**
```typescript
isLoggedIn: boolean
loadState: 'idle' | 'loading' | 'error' | 'success'
appState: AppState        // Full response from GET /employees/me/app-state
activeView: View          // Current screen
loginError: string | null
activeRequest: AdvanceRequest | null  // From appState.activeApplication
kycComplete: boolean
bankComplete: boolean
```

**Key methods:**
```typescript
login(email, password)
logout()
forgotPassword(email)
resetPassword(token, password)
submitAdvanceRequest(amount, purpose)
cancelAdvanceRequest(id)
refreshAppState()           // Re-fetch from /employees/me/app-state
initiatePayment(loanApplicationId)
verifyPayment(orderId, paymentId, signature)
```

**App state refresh:** Called on login and whenever a state-changing action completes (apply, cancel, payment verified). Calls `GET /employees/me/app-state`.

---

## Key Components

### AppShell (`src/components/layout/AppShell.tsx`)
Wraps all authenticated screens. Contains the header and TabBar.

### TabBar
Bottom navigation with 4 tabs: Home, Activity, Advance, Profile. Active tab is inferred from `activeView`.

---

## Data Flow

```
useEmployeeApp → api.ts (fetch calls) → Backend REST API
useEmployeeApp → appState → passed as props to screens
Screens → call methods on useEmployeeApp hook → trigger API → refresh state
```

All API calls go through `src/services/api.ts`. This file exports typed functions for every endpoint. No axios — uses native `fetch`.

---

## Themes

Light and dark mode. Controlled by `useTheme` hook. CSS classes applied at root: `app-root--light` / `app-root--dark`. All color values are CSS custom properties (variables).

---

## Key Files

| File | Purpose |
|---|---|
| `src/screens/EmployeeApp.tsx` | Root view router |
| `src/hooks/useEmployeeApp.ts` | All state + API logic |
| `src/services/api.ts` | All typed fetch functions |
| `src/types/app.ts` | All TypeScript types (AppState, AdvanceRequest, etc.) |
| `src/styles.css` | All CSS — custom properties, layout, component styles |
| `src/config.ts` | API base URL, environment config |
| `src/hooks/useSignedUrl.ts` | Fetches signed URLs for private R2 files |

---

## Onboarding Flow

Employee must complete these steps before the Advance screen unlocks:

1. App activated (password set via email) → handled by backend
2. KYC uploaded → `OnboardingKycScreen`
3. Bank account added → `OnboardingBankScreen`
4. Admin verifies KYC, bank, selfie → reflected in `appState.eligibility.checks`

Until all checks pass, the Advance screen shows a blocker card with a CTA to complete the next step.

---

## Platform Fee Flow (Razorpay)

After employer approval, the employee must pay ₹175:

1. `GET /platform-fees/config` — get amount and Razorpay key
2. `POST /platform-fees/loan-applications/:id/initiate-payment` — create Razorpay order
3. Open Razorpay checkout with `orderId`, `amount`, `key`
4. On success: `POST /platform-fees/verify-payment` with `orderId + paymentId + signature`
5. Backend verifies HMAC signature and moves application to `READY_FOR_DISBURSAL`

The Razorpay checkout JS SDK is loaded via CDN in `index.html`.

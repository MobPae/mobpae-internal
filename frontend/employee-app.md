# Frontend: Employee App (mobpae-app)

**Repo:** `mobpae-app`  
**Stack:** React 19, TypeScript, Vite, custom CSS (no component library)  
**Target:** Mobile browser / Capacitor native (390px shell)

---

## Architecture

The app is a single-page application with a custom view router. There is no React Router — navigation is managed with an `activeView` state variable in `useEmployeeApp`.

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
| `ChangePasswordScreen` (forced) | gate — no view key | First-time forced password set |
| `TermsAcceptanceScreen` | gate — no view key | T&C acceptance gate (once) |
| `DashboardScreen` | `home` (authenticated) | Hero card, eligibility, quick actions |
| `AdvanceScreen` | `advance` | Amount selector, repayment preview, apply CTA |
| `ActivityScreen` | `activity` | Tabs: Active requests / Repayments / History |
| `RepaymentScheduleScreen` | `repayments` | Breakdown of scheduled repayment |
| `ProfileScreen` | `profile` | User profile, settings links |
| `OnboardingKycScreen` | `onboarding-kyc` | Upload KYC documents |
| `OnboardingBankScreen` | `onboarding-bank` | Add bank account |
| `OnboardingDoneScreen` | `onboarding-done` | Onboarding completion screen |
| `NotificationsScreen` | `notifications` | Notification list |
| `HelpScreen` | `help` | FAQ / help topics |
| `LegalScreen` | `legal` | Terms, Privacy Policy links |
| `ChangePasswordScreen` (voluntary) | `change-password` | Update password from Profile |

---

## First-Time Login Gate Flow

New employees are created with a temporary password by the employer. The app enforces two sequential gates before the main dashboard is shown:

```
Login
  ↓ (passwordChanged === false in login response)
ChangePasswordScreen [forced=true]
  — "Log out" link on left; no tab bar; no navigation bypass
  — On success → new tokens issued; termsAccepted checked from response
  ↓ (termsAccepted === false)
TermsAcceptanceScreen
  — "Log out" link top-right; cannot skip
  — On accept → POST /auth/accept-terms → loadEmployee()
  ↓
Dashboard (normal app)
```

**Gate persistence across page refreshes:**  
Both gate flags are persisted in localStorage so a refresh does not bypass the gates:

| Key | Value | Cleared when |
|---|---|---|
| `mobpae_must_change_pwd` | `"1"` | Password successfully changed |
| `mobpae_must_accept_terms` | `"1"` | Terms accepted (POST /auth/accept-terms succeeds) |

Both keys are also cleared on `logout()` and on session expiry.

**T&C check on every app load:**  
`loadEmployee()` reads `termsAccepted` from the `loadAppState()` response — no separate network call. `GET /employees/me` is fetched exactly once, and `termsAccepted` is extracted from that same response alongside all other app data.

---

## Navigation Pattern

No React Router. All navigation goes through `app.setActiveView(viewKey)`.

The `EmployeeApp.tsx` component renders in this order:
```tsx
if (!app.isLoggedIn)           → auth screens (login / forgot-pwd / reset-pwd)
if (app.mustChangePassword)    → ChangePasswordScreen [forced=true]  ← gate 1
if (app.mustAcceptTerms)       → TermsAcceptanceScreen               ← gate 2
if (app.loadState !== 'ready') → skeleton / error screen
→ AppShell with tab bar + current view
```

Tab bar (bottom navigation) is rendered inside `AppShell` and only visible after both gates pass.

---

## Central State Hook: `useEmployeeApp`

Located at `src/hooks/useEmployeeApp.ts`. This is the single source of truth for the entire app.

**Key state:**
```typescript
isLoggedIn: boolean
loadState: 'idle' | 'loading' | 'ready' | 'error'
appState: AppState            // Populated by loadAppState()
activeView: View              // Current screen
mustChangePassword: boolean   // First-time forced password gate
mustAcceptTerms: boolean      // T&C acceptance gate
loginError: string
changingPassword: boolean
changePasswordError: string
eligibility: EligibilityResult | null
kycComplete: boolean
bankComplete: boolean
```

**Key methods:**
```typescript
login(email, password)         // Sets mustChangePassword / mustAcceptTerms if needed
logout()                       // Clears tokens, localStorage flags, all state
changePassword(current, next)  // Forced: issues new tokens + checks termsAccepted
                               // Voluntary: invalidates all sessions → logout
acceptTerms()                  // POST /auth/accept-terms → clears mustAcceptTerms flag
loadEmployee(checkOnboarding?) // Main data fetch; checks T&C from loadAppState result
forgotPassword(email)
resetPassword(token, password)
submitSalaryAdvance(amount)
cancelAdvanceRequest(id)
refresh()                      // Manual pull-to-refresh
```

**`loadAppState()` in `api.ts`** calls `GET /employees/me` once and returns the full `AppState` plus a `termsAccepted: boolean` field. This eliminates a previously separate `checkTermsAccepted()` call that hit `/employees/me` twice on every load.

---

## Key Components

### AppShell (`src/components/layout/AppShell.tsx`)
Wraps all authenticated screens. Contains the header and TabBar.

### TabBar
Bottom navigation with 5 tabs: Home, Activity, Advance, Repayments, Profile. Active tab is inferred from `activeView`.

---

## Data Flow

```
useEmployeeApp → api.ts (fetch calls) → Backend REST API
useEmployeeApp → appState → passed as props to screens
Screens → call methods on useEmployeeApp hook → trigger API → refresh state
```

All API calls go through `src/services/api.ts`. This file exports typed functions for every endpoint. No axios — uses native `fetch`.

---

## AppState Type

```typescript
type AppState = {
  profile: EmployeeProfile;
  dashboard: EmployeeDashboard | null;
  documents: KycDocument[];
  bankAccount: BankAccount | null;
  platformFeeConfig: PlatformFeeConfig | null;
  requests: AdvanceRequest[];
  rawNotifications: AppNotification[];
  peerActivity: PeerActivity | null;
};
```

All notification rendering uses `rawNotifications`. There is no `notifications: string[]` field — that dead field was removed.

---

## Themes

Light and dark mode. Controlled by `useTheme` hook. CSS classes applied at root: `app-root--light` / `app-root--dark`. All color values are CSS custom properties (variables).

---

## Key Files

| File | Purpose |
|---|---|
| `src/screens/EmployeeApp.tsx` | Root view router + gate rendering |
| `src/hooks/useEmployeeApp.ts` | All state + API logic |
| `src/services/api.ts` | All typed fetch functions |
| `src/types/app.ts` | All TypeScript types (AppState, AdvanceRequest, etc.) |
| `src/data/emptyState.ts` | Default AppState used before first load |
| `src/styles.css` | All CSS — custom properties, layout, component styles |
| `src/config.ts` | API base URL, environment config |
| `src/hooks/useSignedUrl.ts` | Fetches signed URLs for private R2 files |
| `src/services/pushNotifications.ts` | Capacitor push notification registration |

---

## Onboarding Flow

Employee must complete these steps before the Advance screen unlocks:

1. First-time password change (gate 1) → `ChangePasswordScreen [forced]`
2. T&C acceptance (gate 2) → `TermsAcceptanceScreen`
3. KYC uploaded → `OnboardingKycScreen`
4. Bank account added → `OnboardingBankScreen`
5. Admin verifies KYC + bank → reflected in `eligibility.eligible`

Until all checks pass, the Advance screen shows a blocker card with a CTA to complete the next step.

---

## Platform Fee Flow (Razorpay)

After employer approval, the employee may need to pay a platform fee:

1. `GET /platform-fees/config` — get amount and Razorpay key
2. `POST /platform-fees/loan-applications/:id/initiate-payment` — create Razorpay order
3. Open Razorpay checkout with `orderId`, `amount`, `key`
4. On success: `POST /platform-fees/verify-payment` with `orderId + paymentId + signature`
5. Backend verifies HMAC signature and moves application to `READY_FOR_DISBURSAL`

The Razorpay checkout JS SDK is loaded via CDN in `index.html`.

---

## Push Notifications (Capacitor)

The app is wrapped with Capacitor for native iOS/Android builds. Push notifications are registered on first authenticated app load via `src/services/pushNotifications.ts`:

1. Request permission via `PushNotifications.requestPermissions()`
2. Register with APNs/FCM via `PushNotifications.register()`
3. On token received: `POST /notifications/device-token` to store in backend
4. On logout: `DELETE /notifications/device-token` to unregister

Firebase is the push provider. The backend `PushNotificationService` wraps firebase-admin and is triggered from `LoanApplicationsService` on status transitions.

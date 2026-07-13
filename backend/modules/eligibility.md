# Module: eligibility

Computes whether an employee is eligible to submit an advance request and what their maximum amount is.

**Service:** `src/eligibility/eligibility.service.ts`

---

## When is it Called?

1. `GET /loan-applications/eligibility` — employee app checks before showing the request form
2. `GET /employees/me/app-state` — included in the full app state response
3. `POST /loan-applications` — final check before accepting a submission (throws if not eligible)

---

## EligibilityResult Shape

```typescript
{
  isEligible: boolean,
  reason: string | null,     // human-readable reason if not eligible
  maxAdvanceAmount: number,  // computed max they can request
  advanceLimit: number,      // admin-set hard limit
  checks: {
    kycComplete: boolean,
    bankVerified: boolean,
    selfieVerified: boolean,
    salaryMinimumMet: boolean,
    tenureMet: boolean,
    noActiveAdvance: boolean,
    cooldownPassed: boolean,
    employerActive: boolean,
    employeeActive: boolean
  }
}
```

---

## Checks Performed

1. **Employee active** — `employmentStatus === ACTIVE`
2. **Employer approved** — `employer.status === APPROVED`
3. **KYC complete** — all required document types have status `VERIFIED`
4. **Bank verified** — `bankAccount.verified === true`
5. **Selfie verified** — `selfieStatus === VERIFIED`
6. **Minimum salary** — `employee.salaryInHand >= config.minimumSalaryInHand`
7. **Minimum tenure** — `joiningDate` is at least `minimumTenureMonths` months ago
8. **No active advance** — no existing application in a non-terminal status
9. **Cooldown passed** — last repayment was more than `cooldownDays` ago

---

## Max Amount Computation

See [Business Rules — Advance Limit Computation](../business-rules.md#2-advance-limit-computation) for the full formula.

In short: takes the minimum of salary-based %, employer override, platform max, and admin-set LoanLimit.

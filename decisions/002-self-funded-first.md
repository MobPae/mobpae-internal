# ADR 002: Self-Fund First, NBFC Partnership Later

**Status:** Accepted  
**Date:** 2024

---

## Context

Salary advance platforms typically need an NBFC or bank partnership to legally lend money. Onboarding an NBFC is a lengthy process involving RBI licensing, due diligence, and term negotiation. This could delay the MVP launch by 6–12 months.

---

## Decision

MobPae will self-fund all advances during the early phase from its own capital. No NBFC is involved.

The schema is designed for future NBFC onboarding:
- `FundingPartner` model exists and is schema-ready
- `LoanApplication.fundingPartnerId` is nullable — `NULL` means self-funded
- When NBFC is onboarded, each application can be attributed to the NBFC via this field

Revenue model during self-funded phase: ₹175 platform fee per approved advance. Interest is also collected but since MobPae bears the capital cost, net economics depend on volume.

---

## Consequences

**Good:**
- Can launch immediately without waiting for NBFC approval
- Full control over the product experience
- Builds repayment track record which helps NBFC due diligence later

**Bad:**
- Capital risk — MobPae's own funds are at risk if employer doesn't pay settlement
- Scale is limited by available capital
- Regulatory grey area — self-funded advances don't require NBFC license but must be structured correctly

**Future path:**
When NBFC is onboarded, set `LoanApplication.fundingPartnerId` for each new application. The NBFC provides capital; MobPae takes platform fee. Interest economics shift to NBFC.

# ADR 004: Two-Step Disbursal (Create Then Disburse)

**Status:** Accepted  
**Date:** 2024

---

## Context

When an admin is ready to send money to an employee, we could trigger the bank transfer in a single step. However, a single-step process is risky: there's no opportunity for a final review before funds move.

---

## Decision

Disbursal is split into two explicit admin steps:

**Step 1: Create Disbursal (`POST /disbursals`)**
- Admin reviews the application and bank details
- System freezes the employee's current bank account details on the disbursal record
- Creates the `Repayment` record with the full breakdown
- Application status → `DISBURSED` / `REPAYMENT_SCHEDULED`

**Step 2: Trigger Transfer (`POST /disbursals/:id/disburse`)**
- Admin physically transfers money (currently via bank portal)
- Admin clicks "Mark as Disbursed" to confirm
- Status → `SUCCESS`, employee notified

---

## Consequences

**Good:**
- Final verification opportunity before money moves
- Bank details are frozen at Step 1 — changes after this point don't affect the disbursal
- If bank transfer fails, the record is already in the system and can be retried
- Audit trail is clear: two separate timestamps (initiatedAt, completedAt)

**Bad:**
- Requires two admin actions instead of one
- More complex state management

**Future:**
When Razorpay Payout API is integrated, Step 2 will be automated — the system will call Razorpay Payouts automatically after Step 1 and mark as disbursed when the webhook confirms delivery.

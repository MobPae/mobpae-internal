# ADR 005: One Active Advance Per Employee Per Cycle

**Status:** Accepted  
**Date:** 2024

---

## Context

Should employees be allowed to take multiple advances simultaneously — e.g., ₹3,000 in week 1 and another ₹2,000 in week 2 of the same month?

---

## Decision

An employee can have at most **one active advance at a time**. This is enforced by the eligibility check `noActiveAdvance`.

"Active" means the application is in any non-terminal status:
- `SUBMITTED`
- `EMPLOYER_APPROVED`
- `AWAITING_PLATFORM_FEE_PAYMENT`
- `READY_FOR_DISBURSAL`
- `DISBURSED`
- `REPAYMENT_SCHEDULED`

The check is controlled via `LoanProductConfig.eligibilityRules.maxRequestsPerCycle = 1`.

---

## Rationale

The advance is recovered from a single payroll. If an employee takes two advances, the employer must recover two amounts from the same paycheck, which complicates payroll processing and increases the risk of partial recovery.

Keeping one advance per cycle also:
- Simplifies the settlement calculation (one line item per employee)
- Reduces default risk (employee's entire recoverable amount is a single known quantity)
- Makes the employee app simpler to explain

---

## Consequences

**Good:**
- Simple settlement structure
- Easier employer payroll integration
- Lower risk

**Bad:**
- Lower revenue potential per employee
- Employee may need more than the single advance allows

**Future:**
May allow a second advance in the same cycle if the combined amount doesn't exceed the limit and the employer explicitly opts in. Would require settlement line items to support multiple entries per employee per cycle.

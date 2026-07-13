# ADR 001: Snapshot Pattern for Loan Applications

**Status:** Accepted  
**Date:** 2024

---

## Context

When an employee submits a salary advance request, several dynamic values are used to compute the repayment:
- Annual interest rate (from `LoanProductConfig`)
- Salary in hand (from `Employee.salaryInHand`)
- Interest-free threshold (from `LoanProductConfig.eligibilityRules`)
- Number of interest days (derived from submission date + payroll cycle)
- Recovery date (derived from employer payroll config)

These values can change over time. The employer might update the employee's salary. The admin might update the product config (change the interest rate). If we read these values at repayment time rather than submission time, the employee would be charged based on different conditions than what they agreed to.

---

## Decision

All rate/salary/date values are frozen on `LoanApplication` at submission time as `snapshot*` fields:

```
snapshotAnnualInterestRate
snapshotInterestFreeThreshold
snapshotProcessingFeeRate
snapshotGstRate
snapshotMaxAdvancePercentage
snapshotSalaryInHand
snapshotInterestDays
snapshotRecoveryDate
snapshotPayrollCycle
```

The repayment breakdown is always computed using these frozen values, never the live values.

---

## Consequences

**Good:**
- Employees always see the exact charges they agreed to at submission
- Rate changes don't affect in-flight applications
- Auditable — you can always reconstruct how any repayment amount was derived
- Legally defensible — no surprise charges

**Bad:**
- `LoanApplication` table has many snapshot columns
- If salary data is wrong at submission time, the repayment is based on incorrect data (but this is correct behavior — the data at submission time is what matters)

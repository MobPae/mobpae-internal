# ADR 003: Platform Fee Per Advance vs Monthly Membership

**Status:** Accepted  
**Date:** 2024

---

## Context

The original MobPae design used a monthly membership model: employees paid a fixed monthly fee (e.g., ₹99/month) to access the salary advance feature regardless of usage.

This created several problems:
- Employees who didn't use the advance still paid the subscription
- High churn if employees didn't get value every month
- Employer relationship didn't naturally drive revenue
- Felt more like a subscription than a financial service

---

## Decision

Replace the membership model with a per-advance platform fee of ₹175.

- Fee is charged once per approved advance
- Paid by the employee via Razorpay after employer approval
- Revenue is directly tied to usage (approvals)
- Aligns MobPae's incentive with employee and employer success

---

## Implementation

The old `Membership` and `MembershipPlanConfig` models are deprecated. All references to membership in code have been removed or replaced.

The new model:
- `LoanApplicationFee` — one per advance, `feeType: PLATFORM_FEE`, `amount: 175`
- Stored in `Settings` table under key `PLATFORM_FEE_CONFIG` for easy configuration
- Paid via Razorpay → webhook confirms payment → application proceeds

---

## Consequences

**Good:**
- Revenue directly tied to value delivered (each approval = ₹175)
- No churn from inactive users
- Simpler mental model for employees: "pay ₹175 to unlock your advance"
- Employer bears no cost (fee is on employee)

**Bad:**
- Revenue is lumpy — only comes when advances are approved
- Lower predictability vs subscription model
- ₹175 is a relatively small amount — needs volume to be meaningful

**Future:**
May add a thin employer-side fee (annual platform access fee per employer) in addition to the per-advance fee to create more predictable revenue.

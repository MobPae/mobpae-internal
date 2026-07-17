# Glossary

Domain terms used across MobPae's codebase, documentation, and product.

---

**Advance / Salary Advance**  
The core product. An employee requests a portion of their earned salary before their official payday. Not a loan in the traditional sense — it's early access to money already earned.

**Advance Limit**  
The maximum ₹ amount a specific employee can request. Set by admin in the `LoanLimit` table. May be lower than what the eligibility formula would otherwise allow.

**Application Number**  
Human-readable unique ID for a loan application. Format: `MP-SA-YYYY-XXXXXXXX` (e.g., `MP-SA-2024-00000001`). The `SA` denotes Salary Advance product type.

**Cooldown Period**  
A waiting period (in days) after a repayment before an employee can apply again. Currently set to 0 (no cooldown). Configurable per product via `eligibilityRules.cooldownDays`.

**Cycle Date**  
The first day of the month in which repayment falls. Used as the key for grouping repayments into a single `EmployerSettlement`. E.g., all advances due in July = cycle date `2024-07-01`.

**Disbursal**  
The act of transferring the approved advance amount to the employee's bank account. A two-step process: admin creates the disbursal record, then triggers the actual transfer.

**Employer Settlement**  
A monthly invoice MobPae sends to an employer. Contains all employees whose repayments are due that payroll cycle. The employer pays MobPae this total; MobPae marks all included repayments as PAID.

**Employment Status**  
Whether an employee is `ACTIVE` (employed, can use platform) or `INACTIVE` (resigned/terminated, cannot apply).

**Funding Partner**  
An NBFC or bank that provides the capital for disbursals. Currently MobPae is self-funded (all `LoanApplication.fundingPartnerId` are NULL). The `FundingPartner` model is a placeholder for future NBFC partnerships.

**Hard Ceiling**  
The absolute maximum percentage of salary any employer can allow employees to advance, regardless of admin or employer overrides. Currently 50% of salary.

**Interest-Bearing Amount**  
The portion of the advance that accrues interest at 36% per annum. Calculated as `principal - interestFreeAmount`. Interest formula: `interestBearingAmount × (36/100) × (interestDays/365)`.

**Interest-Free Amount**  
The portion of the advance with 0% interest. Currently: `min(principal, platformMaxAdvanceAmount)` where `platformMaxAdvanceAmount = ₹5,000`. So the first ₹5,000 of any advance is always interest-free.

**Interest Days**  
The number of days between the advance submission date and the payroll recovery date. Frozen at submission time in `LoanApplication.snapshotInterestDays`. Used in the interest calculation.

**KYC (Know Your Customer)**  
The identity verification process. Employees must upload Aadhaar, PAN, and Salary Slip. Admin manually verifies each document.

**LoanApplicationFee**  
The ₹175 platform fee charged per approved advance. Paid by the employee via Razorpay after employer approval. Also called "platform fee."

**LoanLimit**  
Admin-set per-employee record specifying the maximum advance they're allowed to take. Set separately from the salary-based eligibility calculation.

**LoanProduct**  
A financial product type (e.g., SA = Salary Advance, PL = Personal Loan). Currently only SA is live; others are "coming soon."

**LoanProductConfig**  
Immutable versioned configuration for a product. Contains eligibility rules, pricing rules, and operational rules as JSON. When rates change, a new version is created — the old version is never edited.

**NBFC (Non-Banking Financial Company)**  
A financial institution licensed by RBI to provide loans. MobPae plans to partner with NBFCs to provide the lending capital. Currently self-funded.

**Outstanding Amount**  
On an `EmployerSettlement`: the remaining amount the employer owes MobPae. Decremented as payments arrive. Starts equal to `totalAmount`.

**Payroll Cutoff Date**  
The day of the month after which a request is considered "too late for this cycle" and its repayment is scheduled for the next payroll date. E.g., cutoff = 21st: requests submitted on or after the 21st recover next month.

**Payroll Date**  
The day of the month when the employer pays salaries (e.g., 28th). This is also when MobPae recovers the advance from the employee's salary via the settlement.

**Platform Fee**  
₹175 charged per approved salary advance, paid by the employee via Razorpay. This is MobPae's primary revenue. See [platform-fees.md](./backend/modules/platform-fees.md).

**Principal**  
The base advance amount — the actual money sent to the employee. Does not include interest, fees, or GST.

**Recovery Date**  
The payroll date on which the advance will be deducted from the employee's salary by the employer. Logic: if submission day < cutoff day → this month's payroll date; else → next month's payroll date.

**Repayment**  
A record created at disbursal time. Represents what the employee owes: principal + interest + fees. Linked to an `EmployerSettlement` line item. Status: SCHEDULED → PAID (or OVERDUE).

**Risk Status**  
A signal on the `Employer` model (`GOOD`, `WARNING`, `BLOCKED`) based on their settlement payment history. Admins can update it manually or via the risk check endpoint.

**Self-Funded**  
MobPae provides the advance money from its own capital, not from an NBFC. `LoanApplication.fundingPartnerId = NULL` indicates self-funding.

**Settlement Line Item**  
One row within an `EmployerSettlement` representing a single employee's advance. Contains frozen employee details and the breakdown (principal, interest, etc.).

**Snapshot**  
All rate/salary/date fields frozen on `LoanApplication` at submission time. Prefixed with `snapshot` (e.g., `snapshotAnnualInterestRate`, `snapshotSalaryInHand`). Ensures the repayment calculation is always based on the exact conditions at submission, even if rates change later.

**Total Amount**  
The total an employee owes at repayment: `principal + interest + processingFee + GST`. Stored on the `Repayment` record.

**Versioned Config**  
The pattern where `LoanProductConfig` rows are immutable once active. To change rates: create a new config row, mark it active. The previous version is deactivated but never deleted (for audit purposes).

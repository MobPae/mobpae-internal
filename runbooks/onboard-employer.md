# Runbook: Onboard a New Employer

End-to-end steps to get a new employer live on the MobPae platform.

**Who:** Admin  
**Time:** 30–60 minutes (depends on employer's employee data readiness)

---

## Overview

```
1. Create employer account
2. Configure product settings
3. Employer adds employees
4. Admin verifies employee KYC + bank accounts
5. Admin sets loan limits
6. Employer goes live
```

---

## Step 1: Create the Employer Account

In admin panel: **Employers** → **Add Employer**.

Required fields:
- Company name (legal name)
- Company code (short unique code, e.g. `LTIM` for LTIMindtree — used in settlement numbers)
- Contact person name
- Email (this becomes their login email)
- Phone
- Payroll date (day of month salaries are paid, e.g. 28)
- Payroll cutoff date (day after which requests go to next cycle, e.g. 21)

Or via API:
```json
POST /employers
{
  "companyName": "LTIMindtree",
  "companyCode": "LTIM",
  "contactPerson": "Priya Sharma",
  "email": "priya@ltim.com",
  "phone": "9876543210",
  "payrollDate": 28,
  "payrollCutoffDate": 21
}
```

The employer receives an activation email with a link to set their password.

---

## Step 2: Set Employer Status to Active

After creation, status defaults to `PENDING`. Activate:

```json
PATCH /employers/:id
{ "status": "ACTIVE" }
```

Or in admin panel: open employer → change status to **Active**.

---

## Step 3: Configure Product Settings

Set the maximum advance amount for this employer's employees:

```json
PATCH /employers/:id/product-config
{
  "productType": "SA",
  "maximumAdvanceAmountOverride": 10000,
  "isEnabled": true,
  "requiresEmployerApproval": true
}
```

- `maximumAdvanceAmountOverride`: The absolute ₹ cap for this employer (overrides the % calculation if lower). Set based on your agreement with the employer.
- `requiresEmployerApproval`: Almost always `true` — employer must approve each request.

---

## Step 4: Employer Adds Employees

Send the employer their login credentials. They log into the employer portal and either:

**Single employee:**  
Employees → Add Employee → fill the form

**Bulk employees:**  
Employees → Bulk Add → paste CSV data with columns: `employeeCode, name, email, phone, salaryInHand`

Each employee receives an activation email. They must click the link to set a password and activate their account.

---

## Step 5: Admin Sets Loan Limits for Each Employee

Once employees are created, set the per-employee advance ceiling:

In admin panel: Employees → click employee → Set Loan Limit.

Or via API:
```json
POST /loan-limits
{
  "employeeId": "uuid",
  "maximumEligibleAmount": 5000,
  "maxRequestsPerCycle": 1,
  "cooldownDays": 0
}
```

**What amount to set:** Typically based on salary percentage and agreement with employer. The eligibility formula will further constrain this to 50% of salary at most.

---

## Step 6: Employee Completes Onboarding

Each employee must:
1. Activate account (set password via email link)
2. Upload KYC documents (Aadhaar, PAN, Salary Slip)
3. Add bank account

---

## Step 7: Admin Verifies KYC and Bank

In admin panel: **KYC** section → grouped by employee → verify each document.

**For each employee:**
- Open KYC drawer → view each document via signed URL → click **Verify** or **Reject**
- Open Bank Verification → check account details → click **Verify**

---

## Step 8: Employee Is Now Eligible

Once KYC and bank are all verified AND loan limit is set, the employee's advance button unlocks in the app.

Verify via:
```
GET /employees/me/app-state   (as the employee)
```

Check `eligibility.isEligible = true` and all `checks` are `true`.

---

## Checklist

- [ ] Employer account created and status = ACTIVE
- [ ] Employer product config set (advance amount cap, enabled)
- [ ] Employer has set their password
- [ ] Employees added (single or bulk)
- [ ] All employees have activated accounts
- [ ] KYC verified for each employee
- [ ] Bank accounts verified for each employee
- [ ] Loan limits set for each employee
- [ ] Test: one employee logs in → Advance tab shows eligible

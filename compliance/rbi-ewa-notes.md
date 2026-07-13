# Compliance: RBI & Earned Wage Access (EWA) Notes

> **Disclaimer:** This is an internal reference document, not legal advice. Consult a qualified legal counsel familiar with Indian fintech regulations before making compliance decisions.

---

## What Is Earned Wage Access (EWA)?

Earned Wage Access (also called On-Demand Pay or Salary Advance) lets employees access a portion of their already-earned salary before the official payday. It is fundamentally different from a traditional loan:

- The money is already earned (not borrowed in the traditional sense)
- Recovery is from the employee's own salary (low default risk)
- No credit bureau impact (not reported as a loan)
- Employer is the intermediary and bears the recovery responsibility

---

## Regulatory Landscape in India (as of mid-2024)

### Is NBFC License Required?

This is a grey area. The RBI has not issued specific EWA guidelines as of the time of writing. Current interpretation:

- If MobPae lends from its own balance sheet with intent to earn interest → may require NBFC registration
- If MobPae facilitates an advance from the employer's own funds (employer pre-funds the advance) → may not require NBFC license (salary advance by employer)
- If MobPae partners with an NBFC that does the lending → MobPae operates as a Lending Service Provider (LSP) / NBFC-AA; license requirements differ

**MobPae's current position:** Self-funded, treating advances as prepaid salary amounts. Legal counsel should confirm this structure doesn't require NBFC registration.

### Lending Service Provider (LSP) Framework

If MobPae works with NBFCs:
- Must comply with RBI's Digital Lending Guidelines (September 2022)
- Loan disbursals must go directly to borrower's bank account (no pass-through via platform)
- Key Fact Statement (KFS) must be provided to borrower before disbursal
- Annual Percentage Rate (APR) must be disclosed upfront
- Recovery agents must be RBI-registered

### Digital Lending Guidelines (RBI, 2022)

Key requirements if classified as a Digital Lending App (DLA):
- Borrower consent required before data collection
- No access to phone contacts, media, or microphone for collection purposes
- All loan agreements must be in English and local language
- Must have a nodal grievance officer
- Must be registered on the Sahamati/AA framework if using account aggregator

---

## KYC Requirements

**Mandatory for all users:**
- Aadhaar-based OTP verification (or physical copy upload + admin verification)
- PAN card verification
- Bank account verification (penny drop or cancelled cheque)

**Video KYC (V-CIP):** Not currently implemented. May be required if advance amounts exceed certain thresholds or if NBFC registration is obtained.

**Aadhaar authentication:** Requires UIDAI authorization. Current approach (document upload) is compliant for manual verification. For automated Aadhaar OTP-based KYC, need UIDAI partner license or integration via a licensed KUA (KYC User Agency).

---

## Data Localization

RBI requires financial data of Indian customers to be stored in India. All servers, databases, and storage must be in Indian data centers.

Current setup:
- Database: must be hosted in `ap-south-1` (Mumbai) region or equivalent India region
- Cloudflare R2: bucket must be in `APAC` region (check with Cloudflare for India availability)
- Backend: must be hosted in India (Railway's closest region: Singapore — may need to verify)

> **Action required:** Confirm database and storage are hosted in India before processing real customer data.

---

## Interest Rate Disclosure

The 36% per annum interest rate must be disclosed clearly to employees before they apply. The employee app's advance preview screen shows this rate.

For amounts within the interest-free threshold (first ₹5,000), 0% should be explicitly stated — not just shown as ₹0 interest.

---

## Grievance Redressal

Required under RBI guidelines and Consumer Protection Act:
- Named grievance officer
- Grievance email / phone on website and app
- 30-day resolution SLA
- Escalation path to RBI Ombudsman if unresolved

MobPae's Grievance Redressal Policy is in the Legal Docs folder.

---

## Anti-Money Laundering (AML)

- KYC is the primary AML control
- Suspicious transactions should be reported (large/unusual advance requests)
- PAN is collected for all employees — enables transaction reporting if required
- MobPae's KYC & AML Policy is in the Legal Docs folder

---

## References

- RBI Digital Lending Guidelines (2022): https://rbi.org.in/Scripts/NotificationUser.aspx?Id=12382
- PMLA (Prevention of Money Laundering Act) 2002
- IT Act 2000 + IT (Amendment) Act 2008
- DPDP Act 2023 (Digital Personal Data Protection)

# Product Roadmap

MobPae's evolution from MVP to a full financial wellness platform.

---

## Current State (MVP — Live)

- ✅ Salary advance for salaried employees
- ✅ Employer-backed approval workflow
- ✅ Manual KYC verification (admin)
- ✅ Manual disbursal (bank transfer)
- ✅ ₹175 platform fee via Razorpay
- ✅ Monthly employer settlement
- ✅ Employee app, employer portal, admin panel
- ✅ Cloudflare R2 for KYC/selfie storage

---

## Short Term (Next 3 Months)

**Automated Payouts**
- Integrate Razorpay Payouts API for automated disbursals
- Eliminate the manual bank transfer step
- Admin triggers disbursal; system handles the transfer and marks it complete on webhook

**Digital KYC**
- Integrate a KYC API (e.g., Digio, Setu, Karza) for Aadhaar OTP and PAN verification
- Reduce admin KYC review time from days to minutes
- Auto-verify bank accounts via penny drop or account validation API

**Employer Self-Serve Onboarding**
- Employers can sign up on the website and onboard without admin involvement
- Admin approval required before going live, but data entry is self-serve

**NACH / UPI Autopay for Repayments**
- Set up NACH mandate with employee at onboarding
- Repayment is auto-deducted from salary on payroll date
- Removes dependency on employer manually deducting and paying settlement

---

## Medium Term (3–9 Months)

**NBFC Partnership**
- Onboard one NBFC partner to provide capital
- MobPae earns platform fee; NBFC earns interest
- Reduces MobPae's capital risk; enables larger advance sizes
- Schema is ready: `FundingPartner`, `LoanApplication.fundingPartnerId`

**Multi-Employer Payroll Integrations**
- Direct integration with payroll software (Razorpay Payroll, GreytHR, Darwinbox)
- Pull salary data automatically instead of manual employer input
- Reduces data entry errors and enables real-time salary verification

**Employee Credit Score**
- Track repayment behaviour across advances
- Build internal credit score that informs advance limit
- Reward on-time repayers with higher limits

**Mobile App (React Native)**
- Convert the employee web app to a native iOS + Android app
- Push notifications for application status updates
- Biometric authentication

**Employer Analytics Dashboard**
- Utilization reports: which employees use advances, how often, for how much
- Retention signal: employees who use EWA have lower attrition

---

## Long Term (9–24 Months)

**New Financial Products**
- Personal Loan (PL) — unsecured loans for non-payroll employees
- Credit Line (CL) — revolving credit limit, draw-down as needed
- Health Insurance EMI — employer-facilitated insurance via salary deduction
- All products share the same loan application flow, product config, and settlement infrastructure

**Becoming an NBFC**
- Apply for NBFC license from RBI
- MobPae provides its own capital directly, removing the need for a third-party NBFC
- Higher margin per advance; full control over lending terms

**B2B2C Marketplace**
- Multiple NBFCs / lenders compete on MobPae's platform
- Employers pick a preferred lender
- MobPae earns a marketplace fee per disbursed advance

**Financial Wellness Suite**
- Budgeting tools integrated into the employee app
- Savings goals and automated salary-linked savings
- Tax filing assistance
- Health and life insurance marketplace

---

## What Won't Change

- The employer-backed model — advances always involve employer approval + payroll recovery
- The snapshot pattern — repayment terms are always frozen at submission
- One active advance per cycle — this keeps operations simple and risk low
- Privacy-first — KYC data never sold or shared beyond verification purposes

# Test Accounts (Seed Data)

All accounts are created by running `npx ts-node prisma/seed.ts` in `mobpae-backend`.

---

## Admin

| Field | Value |
|---|---|
| Email | `admin@mobpae.com` |
| Password | `Admin@1234` |
| Portal | http://localhost:5173 |

---

## Employer — NorthStar Technologies

| Field | Value |
|---|---|
| Email | `employer@northstar.mobpae.com` |
| Password | `Demo@1234` |
| Company Code | `NORTHSTAR` |
| Portal | http://localhost:5174 |
| Payroll Date | 28th of each month |
| Payroll Cutoff | 21st of each month |

---

## Employees — NorthStar Technologies

All employees: password = `Demo@1234`  
Email pattern: `{code-lowercase}@northstar.mobpae.com`

| Code | Name | Email | Salary |
|---|---|---|---|
| EMP001 | Arjun Sharma | emp001@northstar.mobpae.com | ₹54,000 |
| EMP002 | Priya Nair | emp002@northstar.mobpae.com | ₹62,000 |
| EMP003 | Kabir Khan | emp003@northstar.mobpae.com | ₹47,000 |
| EMP004 | Sneha Rao | emp004@northstar.mobpae.com | — |
| EMP005 | Rahul Verma | emp005@northstar.mobpae.com | — |
| EMP006 | Ananya Iyer | emp006@northstar.mobpae.com | — |
| EMP007 | Vikram Singh | emp007@northstar.mobpae.com | — |
| EMP008 | Deepa Menon | emp008@northstar.mobpae.com | — |
| EMP009 | Aditya Patel | emp009@northstar.mobpae.com | — |
| EMP010 | Meena Krishnan | emp010@northstar.mobpae.com | — |
| EMP011 | Rohit Gupta | emp011@northstar.mobpae.com | — |
| EMP012 | Lakshmi Subramanian | emp012@northstar.mobpae.com | — |

**Login via:** http://localhost:5175

---

## Razorpay Test Cards

Use these when testing the ₹175 platform fee payment flow. Requires Razorpay test keys (`rzp_test_*`).

| Card Type | Number | CVV | Expiry |
|---|---|---|---|
| Visa (success) | 4111 1111 1111 1111 | Any 3 digits | Any future date |
| Mastercard (success) | 5267 3181 8797 5449 | Any 3 digits | Any future date |
| Failure simulation | 4000 0000 0000 0002 | Any | Any future date |

**UPI (test):** Use `success@razorpay` for instant success, `failure@razorpay` for failure.

**Netbanking:** Select any bank → use the test credentials shown in the modal.

> All test payments go through Razorpay's test environment — no real money moves.

---

## Seeded Loan Application Data

The seed creates one complete loan application lifecycle for demonstration:

- **Employee:** Arjun Sharma (EMP001)
- **Amount:** ₹6,500
- **Status:** REPAYMENT_SCHEDULED
- **Repayment Due:** Next payroll cycle
- **Employer Settlement:** Generated for NorthStar, status GENERATED

This gives you a full end-to-end demo state on fresh seed.

---

## Re-seeding

```bash
cd mobpae-backend
npx prisma migrate reset   # wipes DB
npx ts-node prisma/seed.ts  # recreates all accounts + demo data
```

# Module: pricing

Pure computation module. No database writes. No external calls. All methods are deterministic functions.

**Service:** `src/pricing/pricing.service.ts`

---

## Purpose

`PricingService` is the single source of truth for all financial calculations. It is injected into `LoanApplicationsService` (for preview + snapshot) and `DisbursalsService` (for final repayment breakdown).

---

## Method: `computeSnapshot(salary, productConfig, employerConfig)`

Called at **application submission time**. Freezes pricing parameters.

**Returns:**
```typescript
{
  snapshotAnnualInterestRate: number,
  snapshotInterestFreeThreshold: number,  // ₹ amount below which no interest applies
  snapshotProcessingFeeRate: number,
  snapshotGstRate: number,
  snapshotMaxAdvancePercentage: number
}
```

The `interestFreeThreshold` is computed as:
```
min(salary × platformAdvancePercentage%, platformMaxAdvanceAmount)
```

This means: advances up to the platform's "free" limit pay zero interest. Amounts above it accrue interest at the configured annual rate.

---

## Method: `computeRepaymentBreakdown(disbursedAmount, snapshot)`

Called at **disbursal time** to produce the final repayment record.

**Formula:**
```
interestFreeAmount    = min(principal, snapshot.snapshotInterestFreeThreshold)
interestBearingAmount = principal − interestFreeAmount

interestAmount        = interestBearingAmount
                        × (snapshot.snapshotAnnualInterestRate / 100)
                        × (snapshot.snapshotInterestDays / 365)

processingFee         = principal × snapshot.snapshotProcessingFeeRate
gstAmount             = (interestAmount + processingFee) × snapshot.snapshotGstRate

totalAmount           = principal + interestAmount + processingFee + gstAmount
```

All values are rounded to 2 decimal places (rupees).

**Returns:**
```typescript
{
  interestFreeAmount: number,
  interestBearingAmount: number,
  interestAmount: number,
  processingFee: number,
  gstAmount: number,
  totalAmount: number
}
```

---

## Method: `resolveRecoveryDate(submissionDate, payrollDay, cutoffDay)`

Determines which payroll date the advance will be recovered on.

**Rule:**
```
If submissionDay < cutoffDay  → recover on THIS month's payrollDay
If submissionDay >= cutoffDay → recover on NEXT month's payrollDay
```

**Example:**
```
payrollDay   = 28
cutoffDay    = 21
submissionDate = July 15  → recovery = July 28   (15 < 21)
submissionDate = July 25  → recovery = August 28  (25 ≥ 21)
```

---

## Current Rate Configuration (Seed / Default)

| Parameter | Default Value |
|---|---|
| Annual interest rate | 36% |
| Processing fee rate | 0% |
| GST rate | 0% |
| Interest-free threshold | salary × platformAdvancePercentage% |

These values are configurable per loan product version in `LoanProductConfig.pricingRules`.

---

## Worked Example

```
Employee salary:          ₹30,000
Platform advance limit:   50% of salary
Interest-free threshold:  ₹15,000 (50% × 30,000)

Employee requests:        ₹20,000
Interest days:            15

interestFreeAmount    = min(20,000, 15,000) = 15,000
interestBearingAmount = 20,000 - 15,000     = 5,000
interestAmount        = 5,000 × 0.36 × 15/365 = ₹73.97
processingFee         = 0
gstAmount             = 0
totalRepayment        = 20,000 + 73.97       = ₹20,073.97
```

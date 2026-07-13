# Module: disbursals

Handles the physical payout of approved advances to employee bank accounts.

**Controller:** `src/disbursals/disbursals.controller.ts`  
**Service:** `src/disbursals/disbursals.service.ts`

---

## Endpoints

| Method | Path | Role | Description |
|---|---|---|---|
| POST | `/disbursals` | ADMIN | Create disbursal record for an approved application |
| GET | `/disbursals` | ADMIN | List all disbursals |
| POST | `/disbursals/:id/disburse` | ADMIN | Trigger actual payout |

---

## Two-Step Process

Disbursal is deliberately split into two admin steps:

**Step 1: Create** — Admin reviews the application and creates the disbursal record. This freezes the bank account details at this point in time.

**Step 2: Disburse** — Admin triggers the actual money transfer. This is the point where funds move.

This two-step design allows for a final verification before money moves.

---

## What Happens at Creation (POST /disbursals)

1. Validates application is in `READY_FOR_DISBURSAL` status
2. Validates admin approval exists
3. Creates `Disbursal` record with:
   - `requestedAmount` = employee's requested amount
   - `approvedAmount` = adminApprovedAmount
   - Status: `PENDING`
   - **Frozen bank account details** (accountNumber, IFSC, bankName, holderName)
4. Calls `PricingService.computeRepaymentBreakdown()` with the approved amount + frozen snapshot
5. Creates `Repayment` record with the computed breakdown
6. Updates `LoanApplication` status → `DISBURSED`
7. Creates another status update → `REPAYMENT_SCHEDULED`

**Why freeze bank details?** The employee might change their bank account later. The disbursal record always reflects where money was actually sent.

---

## What Happens at Disburse (POST /disbursals/:id/disburse)

Currently: Admin marks as disbursed manually (internal bank transfer tracked outside the system).

Planned: Razorpay Payout API call or NACH integration.

On success:
- `Disbursal.status` → `SUCCESS`
- `Disbursal.disbursedAmount` set
- `Disbursal.completedAt` set
- Notification sent to employee

---

## Repayment Record Created at Disbursal

The `Repayment` record is created as part of the disbursal flow using `computeRepaymentBreakdown`. Fields:

```
principalAmount       — disbursedAmount (actual money sent)
interestFreeAmount    — portion with 0% interest
interestBearingAmount — portion that accrues interest
interestAmount        — computed interest
processingFee         — computed fee
gstAmount             — computed GST
totalAmount           — what employee owes in total
interestRate          — frozen annual rate %
interestDays          — frozen days from submission to recovery
dueDate               — snapshotRecoveryDate from LoanApplication
status                — SCHEDULED
```

---

## Disbursal Status Flow

```
PENDING → PROCESSING → SUCCESS ✅
                    └→ FAILED ❌ (can retry)
PENDING → CANCELLED ❌
```

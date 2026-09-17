# Module: Fees

**Purpose**: the fee plan for a student for one academic year — the total
owed, not a transaction. One Fee record per Student Academic Enrollment.

## Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Student Academic Enrollment | Lookup → Student Academic Enrollments | Yes | |
| Total Fee | Currency | Yes | set once by office when the enrollment is created |
| Amount Collected | Currency, rollup (sum of Payments.Amount) | — | never manually entered |
| Outstanding Amount | Formula: `Total_Fee - Amount_Collected` | — | never manually entered |
| Payment Status | Picklist, set by Deluge | — | Pending / Partially Paid / Paid |

## Validation rules

- **One fee per enrollment** — (Student Academic Enrollment) unique.
- **Read-only derived fields** — `Amount Collected`, `Outstanding Amount`,
  `Payment Status` are not on the edit layout; they only change via
  [../functions/calculateOutstandingFees.deluge](../functions/calculateOutstandingFees.deluge)
  and
  [../functions/updatePaymentStatus.deluge](../functions/updatePaymentStatus.deluge).

## Relationships

- `Payments.Fee` → this module (one fee, many installments).

## Module: Payments

One installment against a Fee.

| Field | Type | Required | Notes |
|---|---|---|---|
| Fee | Lookup → Fees | Yes | |
| Amount | Currency | Yes | must be > 0 |
| Payment Date | Date | Yes | |
| Mode | Picklist | Yes | Cash / Card / Bank Transfer / UPI |
| Reference No. | Text | No | |

## Validation rules

- **Positive amount** — `Amount > 0`; enforced inside
  [../functions/validatePayment.deluge](../functions/validatePayment.deluge)
  since it needs the related Fee's outstanding amount to also check for
  overpayment in the same pass (a plain field-level "> 0" rule handles the
  first half; the function handles the half that needs a lookup).
- **No overpayment** — `Amount <= Fee.Outstanding Amount` unless the finance
  team explicitly enables the (off by default) overpayment allowance — see
  [../../docs/decisions.md](../../docs/decisions.md).

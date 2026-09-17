# Workflow: Payment Recorded

**Module**: Payments
**Trigger**: before insert (validation) + after insert/edit/delete (recalculation)

## Steps

1. Before insert:
   [../functions/validatePayment.deluge](../functions/validatePayment.deluge)
   — positive amount, no overpayment (unless allowed).
2. After insert/edit/delete:
   [../functions/calculateOutstandingFees.deluge](../functions/calculateOutstandingFees.deluge)
   recomputes `Amount Collected` / `Outstanding Amount` on the parent Fee,
   then calls
   [../functions/updatePaymentStatus.deluge](../functions/updatePaymentStatus.deluge)
   to set `Payment Status`.

Recalculating after delete/edit (not just insert) matters: if office
corrects a mis-entered payment amount or removes a duplicate, the Fee's
totals must reflect that too — otherwise Outstanding Amount silently drifts
from reality, which is exactly what "computed, not manual" is meant to
prevent.

## Error handling

A rejected payment (negative amount, overpayment) never reaches step 2 —
`Fees` totals are only touched by validated Payments.

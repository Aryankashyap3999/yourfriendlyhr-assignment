# Reports: Fees

All built directly on `Fees` fields (`Total Fee`, `Amount Collected`,
`Outstanding Amount`, `Payment Status`) — every one of these is already
kept correct by `calculateOutstandingFees` / `updatePaymentStatus`
(see [../functions/](../functions/)), so the reports are plain native
aggregates with no extra calculation logic of their own.

| Report | Source | Grouping/Filter |
|---|---|---|
| Total Fees | Fees | sum(Total Fee), by Academic Year |
| Amount Collected | Fees | sum(Amount Collected), by Academic Year / Class |
| Outstanding Amount | Fees | sum(Outstanding Amount), by Academic Year / Class |
| Students with Outstanding Fees | Fees | filter `Outstanding Amount > 0`, sorted descending |
| Payment Status Breakdown | Fees | count grouped by Payment Status |

Dashboard: "Fees Overview" — Outstanding Amount by Class, Payment Status
Breakdown (pie), and the Students with Outstanding Fees list for the office
to work from.

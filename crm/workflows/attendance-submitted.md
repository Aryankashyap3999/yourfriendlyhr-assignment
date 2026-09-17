# Workflow: Attendance Submitted

**Module**: Attendance
**Trigger**: before insert (custom function validation, form/API submit)

## Steps

1. [../functions/validateAttendance.deluge](../functions/validateAttendance.deluge)
   — reject if duplicate or the enrollment isn't in the current year.
2. Record created.
3. Percentage is **not** recalculated and stored anywhere here — it's read
   on demand via
   [../functions/calculateAttendancePercentage.deluge](../functions/calculateAttendancePercentage.deluge).
4. After insert: the low-attendance-alert workflow (added in its own phase,
   see [../../docs/decisions.md](../../docs/decisions.md#6-additional-feature-low-attendance-alert))
   calls the same percentage function and notifies the parent if it's below
   threshold.

## Error handling

If `validateAttendance` throws, the form submission fails with that message
shown to the teacher — no partial record is ever created (Deluge validation
functions run before the insert, not after).

# Workflow: Low Attendance Alert

**Module**: Attendance
**Trigger**: after insert (chained after the attendance-submitted workflow
— see [attendance-submitted.md](attendance-submitted.md))

## Steps

```text
Attendance record saved
       ↓
calculateAttendancePercentage(enrollmentId)
       ↓
below Attendance_Alert_Threshold (org setting, default 75%)?
       ↓ yes                                    ↓ no
already alerted today for this enrollment?     stop, nothing to do
       ↓ no                    ↓ yes
getParentStudent -> Primary_Parent.Email        stop, no repeat spam
       ↓
sendmail (Zoho Mail via Deluge)
       ↓
Last_Low_Attendance_Alert_Date = today
```

All of this is [../functions/sendLowAttendanceAlert.deluge](../functions/sendLowAttendanceAlert.deluge),
called once, after the Attendance insert succeeds.

## Configuration

`Attendance_Alert_Threshold` is a single CRM org variable (Setup → Custom
Settings), default `75`. Changing school policy on what counts as "low
attendance" is a one-field edit — no workflow or function changes needed
(Open/Closed in practice, see
[../../docs/architecture.md](../../docs/architecture.md)).

## Scalability

Runs once per attendance record saved (event-driven), not a scheduled scan
over every student every night. Cost is proportional to attendance actually
taken that day, not to total student count — see
[../../docs/decisions.md](../../docs/decisions.md#6-additional-feature-low-attendance-alert).

## Error handling

If the student has no parent email on file, the function returns without
sending and without updating the alert date — it does not throw and does
not block the attendance record itself (attendance was already saved before
this step runs).

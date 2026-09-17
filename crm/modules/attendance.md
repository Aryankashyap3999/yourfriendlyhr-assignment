# Module: Attendance

**Purpose**: one immutable row per student per day. Never edited to reflect
"latest state" — each day's record stands on its own so history is exact.

## Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Student Academic Enrollment | Lookup → Student Academic Enrollments | Yes | carries student + year + class + section together |
| Date | Date | Yes | |
| Status | Picklist | Yes | Present / Absent / Half Day / Leave |
| Marked By | Lookup → Teachers | Yes | |

## Validation rules

- **Uniqueness** — (Student Academic Enrollment, Date) must be unique. A
  platform validation rule enforces this unconditionally; see
  [../functions/validateAttendance.deluge](../functions/validateAttendance.deluge)
  for the pre-check that turns a raw constraint failure into a clear message
  before it ever reaches the database rule.
- **Valid status** — `Status` is a closed picklist, so invalid values are
  rejected by the field type itself; no custom check needed for this part
  (native feature covers it).

## Viewing attendance

- **Individual** — the Attendance related list on a Student Academic
  Enrollment record already shows one student's full history; no custom
  report needed.
- **Class-level** — a standard CRM list view of Attendance filtered by
  `Student Academic Enrollment.Section` and `Date`, grouped by Status,
  covers "how did this class do today" without new code. See
  [../reports/](../reports/) (added in the reports/dashboards phase) for the
  attendance-percentage-by-class report, which does need a calculation.

## Relationships

- Belongs to one Student Academic Enrollment.

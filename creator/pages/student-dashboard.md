# Creator Page: Student Dashboard

One page, one child at a time (with a switcher if a parent has more than
one), rather than five separate pages — everything a parent needs about
their child is one screen with sections, which is simpler to navigate and to
secure (one access check per load, not five).

## Access

Gated by [../security/parent-access-control.md](../security/parent-access-control.md)
before any section below renders.

## Sections

### Profile
- Student Name, Student ID, Status
- Current Class, Current Section, Academic Year
  (from the Student Academic Enrollment where `Academic_Year.Is_Current = true`)

### Attendance
- Current-year attendance percentage
  ([[calculateAttendancePercentage]])
- Recent attendance list (last 30 days: date, status)

### Examination Results
- List of Examinations the student has results for, current academic year
- Per examination: subject-wise marks, and the student's overall %
  ([[calculateStudentPerformance]], mode `student_total`)

### Fees
- Total Fee, Amount Collected, Outstanding Amount, Payment Status
- Payment History: list of Payments (date, amount, mode, reference)

## Data source

All of the above is fetched from CRM at page-load/section-load time via the
integration functions in
[../integrations/](../integrations/) (added in the CRM-Creator integration
phase) — no data is duplicated into a Creator-side form/report; Creator only
renders what CRM returns for the already-authorized student.

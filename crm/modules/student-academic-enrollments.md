# Module: Student Academic Enrollments

**Purpose**: the historical backbone of the whole system. One row per
student per academic year — this is what makes "current class/section"
answerable without ever overwriting a prior year's placement.

## Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Student | Lookup → Students | Yes | |
| Academic Year | Lookup → Academic Years | Yes | |
| Class | Lookup → Classes | Yes | |
| Section | Lookup → Sections | Yes | |
| Roll Number | Text | No | |
| Status | Picklist | Yes | Ongoing / Promoted / Repeated / Left |
| Last Low Attendance Alert Date | Date | No | set by the low-attendance-alert workflow; prevents alerting the same parent more than once a day. Naturally resets each year since it lives on the year's enrollment record. |

## Validation rules

- **One enrollment per student per year** — (Student, Academic Year) must
  be unique. This is the rule that guarantees history is never overwritten:
  moving a student to a new year always means a **new** enrollment record,
  never editing last year's.
- **Section belongs to Class** — `crm/validations/academic-assignment.md`:
  reject if `Section.Class != Class`, so a student can't end up in "Class 9"
  with a section that actually belongs to "Class 10".

## Derived: "current" enrollment

There is no `Is Current` field here — a student's current placement is
simply the enrollment whose `Academic Year.Is Current = true`. Any report or
Creator page that needs "the current class/section" filters on that, rather
than the system maintaining a redundant current-state copy.

## Relationships

- `Attendance.Student Academic Enrollment` → this module.
- `Examination Results.Student Academic Enrollment` → this module.
- `Fees.Student Academic Enrollment` → this module.

# Data Model

## Guiding rule

> Current state must not destroy historical state.

A student's class/section changes every academic year. If we stored
`current_class` / `current_section` directly on the `Students` record and
overwrote it each year, we would lose every prior year's placement. Instead,
class/section/year live on a separate **Student Academic Enrollment** record,
one per student per academic year. "Current" is just the enrollment row
whose academic year is marked active — nothing is overwritten.

The same pattern applies anywhere history matters: Attendance and Examination
Results are never updated in place to reflect "the latest state" — each is an
immutable row keyed by the day/exam it belongs to.

## Entities

### Leads (standard CRM module)
Admission enquiry, before a decision is made.

| Field | Notes |
|---|---|
| Enquiry Source | Webform, Walk-in, Referral |
| Student Name (proposed) | |
| Parent Name / Phone / Email | |
| Class Applied For | lookup-like picklist to `Classes` |
| Lead Status | New / Contacted / Follow-up / Converted / Lost |

### Admissions (custom)
The decision workflow, separate from Leads so Lead nurturing logic doesn't
get tangled with admission-decision logic (single responsibility).

| Field | Notes |
|---|---|
| Lead | lookup -> Leads |
| Academic Year Applied For | lookup -> Academic Years |
| Class Applied For | lookup -> Classes |
| Decision | Pending / Confirmed / Rejected |
| Decision Date | |
| Rejection Reason | text, required if Decision = Rejected |
| Student (created) | lookup -> Students, populated on confirmation |

### Students (custom)
The permanent identity record. Deliberately thin — academic placement is
**not** here, it lives in Student Academic Enrollments.

| Field | Notes |
|---|---|
| Student ID | unique, system-generated, format `STU-<YYYY>-<00001>` — see [[generateStudentId]] in workflows.md |
| First / Last Name | |
| Date of Birth | |
| Gender | |
| Admission Date | |
| Status | Active / Inactive / Alumni / Withdrawn |
| Primary Parent | lookup -> Parents/Guardians |

### Parents / Guardians (custom)
| Field | Notes |
|---|---|
| Name, Phone, Email | |
| Creator User Email | the identity a Creator login is matched against — see [docs/security.md](security.md) |
| Students | related list (one parent -> many students; a student can have more than one linked guardian via a join, kept as a simple multi-lookup since the school does not need many-to-many metadata beyond "who can see this child") |

### Academic Years (custom)
| Field | Notes |
|---|---|
| Name | e.g. `2026-27` |
| Start Date / End Date | |
| Is Current | boolean, exactly one year is current at a time |

### Classes (custom)
| Field | Notes |
|---|---|
| Name | e.g. `Class 9` |
| Academic Year | lookup -> Academic Years (a class is defined per year so "Class 9 for 2026-27" can have different sections/capacity than the prior year without touching history) |

### Sections (custom)
| Field | Notes |
|---|---|
| Name | e.g. `A` |
| Class | lookup -> Classes |
| Class Teacher | lookup -> Teachers |

### Subjects (custom)
| Field | Notes |
|---|---|
| Name | |
| Class | lookup -> Classes (a subject is offered for a specific class) |
| Max Marks (default) | used unless an Examination overrides it |

### Teachers (custom)
| Field | Notes |
|---|---|
| Name, Phone, Email | |
| Subjects Taught | related list -> Teaching Assignments |

### Teaching Assignments (custom)
Join record so a teacher can teach many subject/section combinations without
denormalizing subject lists onto Teachers or teacher lists onto Sections.

| Field | Notes |
|---|---|
| Teacher | lookup -> Teachers |
| Section | lookup -> Sections |
| Subject | lookup -> Subjects |

### Student Academic Enrollments (custom)
The historical backbone. One row per student per academic year.

| Field | Notes |
|---|---|
| Student | lookup -> Students |
| Academic Year | lookup -> Academic Years |
| Class | lookup -> Classes |
| Section | lookup -> Sections |
| Roll Number | |
| Status | Ongoing / Promoted / Repeated / Left |

Uniqueness rule: one enrollment per (Student, Academic Year). "Current
class/section" for a student = the enrollment where Academic Year.Is Current
= true. Every other year's enrollment stays untouched forever.

### Attendance (custom)
| Field | Notes |
|---|---|
| Student Academic Enrollment | lookup -> Student Academic Enrollments (carries student + year + class + section together, so attendance is always tied to the correct academic context without duplicating those fields) |
| Date | |
| Status | Present / Absent / Half Day / Leave |
| Marked By | lookup -> Teachers |

Uniqueness rule: one record per (Student Academic Enrollment, Date) — see
[[validateAttendance]] in workflows.md.

### Examinations (custom)
| Field | Notes |
|---|---|
| Name | e.g. `Mid-Term 2026` |
| Academic Year | lookup -> Academic Years |
| Class | lookup -> Classes |
| Exam Date | |

### Examination Results (custom)
| Field | Notes |
|---|---|
| Examination | lookup -> Examinations |
| Student Academic Enrollment | lookup -> Student Academic Enrollments |
| Subject | lookup -> Subjects |
| Max Marks | defaults from Subject, editable per exam |
| Marks Obtained | `0 <= Marks Obtained <= Max Marks` |

Uniqueness rule: one record per (Examination, Student Academic Enrollment,
Subject) — see [[validateExamMarks]] in workflows.md.

### Fees (custom)
The fee plan for a student for an academic year — not a payment.

| Field | Notes |
|---|---|
| Student Academic Enrollment | lookup -> Student Academic Enrollments |
| Total Fee | manually set by office once per year |
| Amount Collected | rollup, computed from Payments — never entered manually |
| Outstanding Amount | formula: `Total Fee - Amount Collected` |
| Payment Status | Pending / Partially Paid / Paid — computed, not manual |

### Payments (custom)
An installment against a Fee.

| Field | Notes |
|---|---|
| Fee | lookup -> Fees |
| Amount | must be > 0, and cannot push Amount Collected above Total Fee |
| Payment Date | |
| Mode | Cash / Card / Bank Transfer / UPI |
| Reference No. | |

## Relationship summary

```text
Academic Year --< Classes --< Sections
Sections --< Teaching Assignments >-- Teachers
Sections --< Teaching Assignments >-- Subjects

Parents/Guardians --< Students
Students --< Student Academic Enrollments >-- Academic Year/Class/Section

Student Academic Enrollment --< Attendance
Student Academic Enrollment --< Examination Results >-- Examinations
Student Academic Enrollment --< Fees --< Payments

Leads --< Admissions >-- Students (on confirmation)
```

## Derived / calculated values

These are computed by Deluge, never entered manually (Open/Closed +
Dependency Inversion in practice — the calculation rule lives in one
function, not copy-pasted at every entry point):

- Attendance percentage — [[calculateAttendancePercentage]]
- Exam performance (subject/class average, student total %) — [[calculateStudentPerformance]]
- Fee collected / outstanding / status — [[calculateOutstandingFees]], [[updatePaymentStatus]]

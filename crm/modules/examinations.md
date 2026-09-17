# Module: Examinations

**Purpose**: one exam event for a class in a year (e.g. `Mid-Term 2026` for
`Class 9`, `2026-27`).

## Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Name | Text | Yes | |
| Academic Year | Lookup → Academic Years | Yes | |
| Class | Lookup → Classes | Yes | |
| Exam Date | Date | Yes | |

## Relationships

- `Examination Results.Examination` → this module.

## Module: Examination Results

Marks for one student, one subject, one exam.

| Field | Type | Required | Notes |
|---|---|---|---|
| Examination | Lookup → Examinations | Yes | |
| Student Academic Enrollment | Lookup → Student Academic Enrollments | Yes | |
| Subject | Lookup → Subjects | Yes | |
| Max Marks | Number | Yes | defaults from `Subject.Max_Marks_default`, editable per exam |
| Marks Obtained | Number | Yes | `0 <= Marks Obtained <= Max Marks` |

## Validation rules

- **Uniqueness** — (Examination, Student Academic Enrollment, Subject) must
  be unique; platform rule plus the explicit pre-check in
  [../functions/validateExamMarks.deluge](../functions/validateExamMarks.deluge)
  (same belt-and-suspenders pattern as attendance — see
  [../../docs/decisions.md](../../docs/decisions.md#4-duplicate-prevention-done-at-two-levels)).
- **Marks range** — enforced inside `validateExamMarks` since it depends on
  the per-record `Max Marks`, not a fixed constant a simple field-level rule
  could express.
- **Subject validity** — reject if `Subject.Class != Examination.Class`, so
  a result can't be recorded for a subject the exam's class doesn't teach.
  Also inside `validateExamMarks` (same call, one round-trip, rather than a
  separate validation rule needing its own record fetch).

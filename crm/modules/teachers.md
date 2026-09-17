# Module: Teachers

**Purpose**: staff who teach and mark attendance/exams.

## Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Name | Text | Yes | |
| Phone | Phone | No | |
| Email | Email | Yes | must be unique — used to resolve which CRM user marks attendance |

## Relationships

- `Sections.Class Teacher` → this module.
- `Teaching Assignments.Teacher` → this module.

## Module: Teaching Assignments

Join module: which Teacher teaches which Subject in which Section. Kept
separate rather than a multi-select field on Teachers or Sections so a
teacher can teach several subject/section combinations without denormalizing
lists on either side.

| Field | Type | Required | Notes |
|---|---|---|---|
| Teacher | Lookup → Teachers | Yes | |
| Section | Lookup → Sections | Yes | |
| Subject | Lookup → Subjects | Yes | |

**Validation**: (Teacher, Section, Subject) must be unique, and `Subject.Class`
must equal `Section.Class` — a teacher can't be assigned a subject that
doesn't belong to that section's class.

# Module: Students

**Purpose**: the permanent identity record. Deliberately excludes
class/section/year — that's [Student Academic Enrollments](student-academic-enrollments.md),
so a year change never overwrites history.

## Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Student ID | Text | Yes | unique, system-generated (`STU-2026-00001`), never manually entered — see [generateStudentId](../functions/generateStudentId.deluge) |
| First Name | Text | Yes | |
| Last Name | Text | Yes | |
| Date of Birth | Date | Yes | |
| Gender | Picklist | No | |
| Admission Date | Date | Yes | |
| Status | Picklist | Yes | Active / Inactive / Alumni / Withdrawn |
| Primary Parent | Lookup → Parents/Guardians | Yes | |

## Validation rules

- **Student ID uniqueness** — platform-level unique field; also never
  exposed as editable on the layout so staff cannot type one in and collide
  with the generator.

## Relationships

- `Student Academic Enrollments.Student` → this module (one student, many
  years).
- `Parents/Guardians.Students` → this module (reverse of Primary Parent).

## Module: Student Academic Enrollments

See [student-academic-enrollments.md](student-academic-enrollments.md).

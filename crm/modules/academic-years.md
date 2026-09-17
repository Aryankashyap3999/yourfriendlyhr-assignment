# Module: Academic Years

**Purpose**: the time anchor everything else (classes, enrollments,
attendance, exams, fees) hangs off. Nothing else changes "current" state by
overwriting a record — a new Academic Year is added, and old ones stay.

## Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Name | Text | Yes | e.g. `2026-27`, unique |
| Start Date | Date | Yes | |
| End Date | Date | Yes | must be after Start Date |
| Is Current | Checkbox | Yes | exactly one record true at a time |

## Validation rules

- **Unique Name** — platform-level unique field.
- **Single current year** — `crm/validations/single-current-academic-year.md`:
  on save, if `Is Current = true`, unset it on every other Academic Year
  record. Implemented as a workflow (`On Create/Edit` → `Is Current` becomes
  true) calling a one-line Deluge update, not a validation rule, since it's
  a correction action rather than a rejection.

## Relationships

- `Classes.Academic Year` → this module (one year has many classes).

# Module: Classes

**Purpose**: a grade level, scoped to one academic year (so "Class 9" in
2025-26 and "Class 9" in 2026-27 are different records — capacity, sections
or subjects can differ year to year without touching last year's data).

## Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Name | Text | Yes | e.g. `Class 9` |
| Academic Year | Lookup → Academic Years | Yes | |

## Validation rules

- **Unique per year** — (Name, Academic Year) combination must be unique;
  enforced via a validation rule so two "Class 9" rows can't be created for
  the same year by mistake.

## Relationships

- `Sections.Class` → this module.
- `Subjects.Class` → this module.

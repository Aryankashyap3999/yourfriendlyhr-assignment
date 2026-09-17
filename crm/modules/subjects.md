# Module: Subjects

**Purpose**: what's taught for a given Class. Scoped to Class (not Section)
since all sections of a class take the same subjects in this school.

## Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Name | Text | Yes | e.g. `Mathematics` |
| Class | Lookup → Classes | Yes | |
| Max Marks (default) | Number | Yes | default ceiling for exam results; an Examination Result can override per-exam if a specific exam changes it |

## Validation rules

- **Unique per class** — (Name, Class) must be unique.

## Relationships

- `Teaching Assignments.Subject` → this module.
- `Examination Results.Subject` → this module.

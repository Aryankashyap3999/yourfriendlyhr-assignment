# Module: Sections

**Purpose**: a division within a Class (e.g. `9-A`, `9-B`).

## Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Name | Text | Yes | e.g. `A` |
| Class | Lookup → Classes | Yes | |
| Class Teacher | Lookup → Teachers | No | |
| Capacity | Number | No | optional, used only for admission section allotment |

## Validation rules

- **Unique per class** — (Name, Class) must be unique.

## Relationships

- `Student Academic Enrollments.Section` → this module.
- `Teaching Assignments.Section` → this module.

# Module: Admissions

**Purpose**: the decision workflow, kept separate from Leads so lead
nurturing (follow-ups, contact attempts) doesn't get tangled with the actual
confirm/reject decision and what it produces.

## Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Lead | Lookup → Leads | Yes | |
| Academic Year Applied For | Lookup → Academic Years | Yes | copied from Lead on create |
| Class Applied For | Lookup → Classes | Yes | copied from Lead on create |
| Decision | Picklist | Yes | Pending (default) / Confirmed / Rejected |
| Decision Date | Date | No | set automatically when Decision changes from Pending |
| Rejection Reason | Text | No | required if Decision = Rejected |
| Student (created) | Lookup → Students | No | populated by [`createStudentFromAdmission`](../functions/createStudentFromAdmission.deluge) on confirmation, read-only |

## Validation rules

- **Rejection reason required** — reject save if `Decision = Rejected` and
  `Rejection Reason` is blank.
- **Decision is one-way** — reject edit if `Decision` is changing away from
  `Confirmed` or `Rejected` back to `Pending` (a decision, once made, should
  be corrected by creating a fresh record if genuinely wrong, not silently
  flipped — keeps the audit trail honest).

## Workflow

See [../workflows/admission-confirmed.md](../workflows/admission-confirmed.md)
for what happens when `Decision` becomes `Confirmed`.

## Relationships

- `Students` (reverse of `Student (created)`).

# Module: Parents / Guardians

**Purpose**: the identity a Creator login is matched against, and the
target of low-attendance alerts.

## Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Name | Text | Yes | |
| Phone | Phone | No | |
| Email | Email | Yes | unique; must match the email the parent logs into Creator with |
| Students | Multi-lookup / related list → Students | Yes | which children this parent can see |

A student can list more than one guardian (e.g. both parents) simply by both
Parent records including that student — this is a many-to-many the module
handles natively via the multi-lookup, without a separate join module, since
no extra metadata (like "primary guardian") is required beyond
`Students.Primary Parent` already on the Students module.

## Relationships

- `Students.Primary Parent` → this module (single, for the "who do we call
  first" case).
- This module's `Students` field → the reverse, for "who can see this child"
  (used by [[getParentStudent]] — see [../functions/getParentStudent.deluge](../functions/getParentStudent.deluge) and [../../docs/security.md](../../docs/security.md)).

# Architecture

## System overview

```text
                     ┌─────────────────────────────┐
                     │          Zoho CRM            │
                     │   (source of truth for all   │
                     │     school data & staff)     │
                     │                               │
                     │  Leads → Admissions → Students│
                     │  Academic structure           │
                     │  Attendance / Exams / Fees     │
                     │  Deluge functions & workflows │
                     └───────────────┬───────────────┘
                                     │ Deluge / Zoho CRM API
                                     │ (read-mostly, on demand)
                     ┌───────────────▼───────────────┐
                     │        Zoho Creator            │
                     │   (parent-facing application)  │
                     │  Login → resolve Parent →      │
                     │  fetch only that parent's      │
                     │  student's data from CRM       │
                     └───────────────┬───────────────┘
                                     │
                                  Parent
```

CRM is the only place data is written for school operations. Creator never
maintains its own copy of student/attendance/exam/fee data — it queries CRM
on demand through Deluge (`invokeurl` / CRM integration tasks). This avoids a
sync mechanism entirely: there is nothing to keep in sync because there is
only one copy of the data.

## Why Zoho-only (no custom backend/frontend)

Zoho CRM already provides: modules with relationships, validation rules,
workflow automation, role/profile based sharing, and reporting/dashboards.
Zoho Creator already provides: a small app UI, its own auth, and native CRM
connectivity. A custom Node/React/DB stack would reimplement all of this and
add a synchronization problem that doesn't otherwise exist. See
[decisions.md](decisions.md) for the explicit trade-off record.

## Business logic layering

Every automated flow follows the same shape, so logic never piles up into
one giant function:

```text
Trigger (form submit / field update / schedule)
       ↓
Validation           (reject bad input early, e.g. validateAttendance)
       ↓
Business logic        (pure calculation, e.g. calculateAttendancePercentage)
       ↓
CRM operation          (create/update the record)
       ↓
Automation/notification (only if the business rule says so, e.g. low-attendance alert)
```

Each stage is a separate, named Deluge function under `crm/functions/`.
Workflows (`crm/workflows/`) wire triggers to these functions; they do not
contain business logic themselves. This is what keeps the system open to
extension (Open/Closed): changing the low-attendance threshold or the marks
validation range means editing one function or one config value, not hunting
through every workflow that touches attendance or marks.

## SOLID, applied where it earns its keep

- **SRP** — `validateAttendance`, `calculateAttendancePercentage`,
  `validateExamMarks`, etc. each do exactly one thing (see
  [workflows.md](workflows.md)). No `processEverything()`.
- **OCP** — thresholds and limits (attendance alert %, negative-marks rule)
  are configuration values read by the function, not literals scattered
  across every workflow that needs them.
- **LSP** — not forced. There is no class hierarchy in Deluge worth
  inventing one for; skipped rather than manufactured.
- **ISP** — CRM modules stay narrow (Fees vs Payments are separate modules,
  not one bloated "Finance" module mixing plan and transactions).
- **DIP** — Creator pages call a small set of integration functions
  (`getParentStudent`, `getStudentSnapshot`) rather than embedding raw CRM
  API/report queries in every page; the page doesn't need to know how CRM
  is queried, only what it gets back.

## What was deliberately not built

- No caching layer — CRM query volume from ~a few hundred parents checking a
  handful of pages is nowhere near Zoho API limits. Add one only if usage
  data says otherwise.
- No custom auth — Creator's own login + a Parent lookup by email is
  sufficient; building a separate auth system would duplicate Creator's own
  security model for no benefit.
- No message queue / sync job — CRM is the only writer, Creator is read-only
  against it, so there is nothing to queue or reconcile.

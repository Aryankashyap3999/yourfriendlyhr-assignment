# CRM Connection

Zoho Creator has a native **CRM integration task** (Deluge:
`zoho.crm.getRecords` / `invokeUrl` against CRM, or the built-in
"CRM Connection" component) that lets a Creator function call directly into
the CRM org, authenticated once via an OAuth connection configured in
Creator's environment (no API key/secret in this repo — configured in the
Zoho org's Connections page).

## Setup

1. Creator → Connections → add a CRM connection scoped to
   `ZohoCRM.modules.ALL` (read) for the modules this app reads:
   `Parents_Guardians`, `Students`, `Student_Academic_Enrollments`,
   `Attendance`, `Examination_Results`, `Fees`, `Payments`.
2. Reference that connection name from Creator functions via
   `zoho.crm.getRecords(..., connection="<connection-name>")`.

No token, secret, or org id is committed here — the connection is
established through Zoho's own OAuth flow in the Creator UI.

## Why this, not a webhook/sync job

Creator queries CRM live, on each page/section load, rather than CRM
pushing data into a Creator-side copy. There is one copy of the data (in
CRM); Creator is a live read window into it. This means:

- a CRM edit is visible in Creator on the next page load — no propagation
  delay to reason about, and no sync job that can fail or drift.
- nothing to reconcile if a sync run is missed, because there's no sync run.

This scales fine for a single school's expected traffic (parents checking a
handful of pages, not high-frequency polling); if usage ever demands lower
latency than a live query provides, the fix is scoped and cached queries in
`creator/functions/`, not a new sync architecture — see
[../../docs/decisions.md](../../docs/decisions.md).

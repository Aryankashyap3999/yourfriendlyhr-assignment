# CRM ↔ Creator Integration

## What's stored where

| Data | Stored in | Displayed in |
|---|---|---|
| Leads, Admissions | CRM only | CRM only (staff) |
| Students, Parents, Academic structure | CRM only | CRM (staff, full), Creator (parent, own child only) |
| Attendance | CRM only | CRM (staff, all), Creator (parent, own child, % + recent) |
| Examination Results | CRM only | CRM (staff, all), Creator (parent, own child) |
| Fees / Payments | CRM only | CRM (staff, all), Creator (parent, own child) |

Creator holds **no** persisted copy of any of the above — every figure a
parent sees is fetched from CRM at view time. See
[creator/integrations/crm-connection.md](../creator/integrations/crm-connection.md)
for how.

## How Creator retrieves CRM data

```text
Creator page (student-dashboard.md)
       ↓ calls
creator/functions/getStudentSnapshot.deluge
       ↓ 1. checks authorization via
crm/functions/getParentStudent.deluge (isParentAuthorizedForStudent)
       ↓ 2. if authorized, fetches via the CRM connection
zoho.crm.getRecords(...) / zoho.crm.functions(...)
       ↓
Page renders the returned snapshot
```

## How parent-to-student mapping works

See [security.md](security.md) — `Parents_Guardians.Email` matched against
the Creator login email, `Parents_Guardians.Students` giving the authorized
set. One definition, read live by both the authorization check and the data
fetch.

## How CRM changes become available in Creator

Immediately, on next page/section load — there is no batch sync, cache
warm-up, or webhook to wait on. An office staff member editing a Payment in
CRM is reflected the next time that parent opens (or refreshes) their fee
section, because that section's query runs fresh, against CRM, every time.

## Failure handling

If the CRM connection call fails (network/API error), `getStudentSnapshot`
lets that exception surface to the page, which shows a
"couldn't load your child's information right now, please try again"
state — it does not silently show stale or partial data as if it were
current.

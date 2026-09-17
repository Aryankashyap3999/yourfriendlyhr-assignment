# Creator: Parent Access Control

Enforced in Deluge, on the server side of every Creator page that shows a
specific student's data — never left to page/UI logic alone.

## Where it's wired in

Each page in [../pages/student-dashboard.md](../pages/student-dashboard.md)
runs this on load, before any CRM data-fetch for the requested student:

```text
on page load, given studentId from page state:
    parentEmail = zoho.loginuser
    authorized = getParentStudent.isParentAuthorizedForStudent(parentEmail, studentId)
    if not authorized:
        set page state -> "access_denied"
        return   // no further CRM calls happen
    // else continue to fetch and render
```

## Picking the default student

On first login (no `studentId` in page state yet), the page calls
`getParentStudent.getStudentsForParent(zoho.loginuser)`:

- zero students → show "no linked student found, contact the school office"
- one student → load that student directly, no picker shown
- more than one (siblings) → show a simple student switcher populated from
  that same list — the switcher's options are exactly the authorized list,
  so there's no way to select an id outside it from the UI, and the load
  step above still re-checks regardless.

## What this deliberately does not do

No Creator-side table of "parent -> student" is maintained — every check
calls the CRM function live. At the scale of a single school (a few hundred
parents, one check per page load) this is well within normal CRM API usage;
adding a cache would be solving a load problem that doesn't exist yet.

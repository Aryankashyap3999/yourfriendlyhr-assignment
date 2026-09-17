# Parent Data Security

## The mapping

```text
Creator login (email)
       ↓  matched against
Parents/Guardians.Email
       ↓  Students multi-lookup field
Student(s) this parent may see
       ↓
Academic / Attendance / Examination / Fee data for those students only
```

`Parents/Guardians.Email` is the same email a parent logs into the Creator
app with. There is no separate "Creator user id" mapping table — Creator's
own logged-in user email *is* the key into `Parents_Guardians`, so there is
exactly one place the parent-to-student link is defined
([crm/modules/parents-guardians.md](../crm/modules/parents-guardians.md)),
not a copy in Creator and a copy in CRM.

## Enforcement — not just hiding UI

The rule from the assignment is explicit: a parent must be **denied**, not
just kept from seeing a button. Every Creator data-fetch for a specific
student goes through this sequence, always server-side (Deluge), never
decided in client-side page logic:

```text
Creator page loads for logged-in user (zoho.loginuser)
       ↓
studentId = page parameter (which child's data is being requested)
       ↓
isParentAuthorizedForStudent(zoho.loginuser, studentId)
       ↓ false                              ↓ true
Deny — return no data, show               Proceed — fetch and
"not authorized" state                    render that student's data
```

[crm/functions/getParentStudent.deluge](../crm/functions/getParentStudent.deluge)
(`isParentAuthorizedForStudent`) is the one function this check always
calls — see [creator/security/parent-access-control.md](../creator/security/parent-access-control.md)
for exactly where it's wired into the Creator page.

## Why this is enough, and why nothing more was added

- A parent guessing/editing a URL parameter to another student's id still
  hits `isParentAuthorizedForStudent` and gets denied — the check doesn't
  trust the parameter, it verifies it.
- No separate Creator-side copy of the parent-student mapping exists to
  drift out of sync with CRM — there's one CRM lookup field, read live.
- A full custom auth/session system was not built: Creator's own login
  already authenticates the user; the only piece this system owns is
  *authorization* (which student), not *authentication* (who is this
  person) — building the latter again would duplicate what Creator
  provides natively.

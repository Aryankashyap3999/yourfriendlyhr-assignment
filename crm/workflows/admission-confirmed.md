# Workflow: Admission Confirmed

**Module**: Admissions
**Trigger**: field update — `Decision` changes to `Confirmed`

## Steps

1. Validation (handled by the `Rejection Reason required` / `Decision is
   one-way` rules on the module itself — see [../modules/admissions.md](../modules/admissions.md)).
2. Call [../functions/createStudentFromAdmission.deluge](../functions/createStudentFromAdmission.deluge)
   with the Admission record ID.
   - internally resolves/creates the Parent via
     [../functions/getOrCreateParent.deluge](../functions/getOrCreateParent.deluge)
   - internally generates the Student ID via
     [../functions/generateStudentId.deluge](../functions/generateStudentId.deluge)
   - creates the Student, the first Student Academic Enrollment, links
     `Admissions.Student`, and sets `Leads.Lead_Status = Converted`
3. Error handling: if `createStudentFromAdmission` throws (e.g. no Section
   with capacity found for the applied-for Class), the workflow's failure
   is surfaced on the Admission record via a CRM workflow alert to the
   Admissions team queue — the Decision is not silently left in an
   inconsistent state, and staff must resolve the missing section before
   re-triggering (re-saving the record) rather than getting a half-created
   Student.

## Workflow: Admission Rejected

**Trigger**: field update — `Decision` changes to `Rejected`

```text
Leads.Lead_Status -> Lost
```

No Deluge needed — a single field update action covers it (native workflow
action, not a function).

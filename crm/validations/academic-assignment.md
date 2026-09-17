# Validation: Academic assignment consistency

Prevents a Student Academic Enrollment (or a Teaching Assignment) from
pairing a Section with the wrong Class.

## Rule 1 — Enrollment section must match class

On `Student Academic Enrollments` create/edit:

```text
reject if Section.Class.id != Class.id
message: "Selected section does not belong to the selected class."
```

## Rule 2 — Teaching assignment subject must match section's class

On `Teaching Assignments` create/edit:

```text
reject if Subject.Class.id != Section.Class.id
message: "Selected subject is not offered for this section's class."
```

Both are implemented as standard CRM **validation rules** (not Deluge
functions) — they're a pure field-comparison guard with no external calls or
calculation, so the platform's native validation rule is the simplest
correct tool (see the ladder: native feature before custom code).

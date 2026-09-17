# School Management System (Zoho CRM + Creator)

A school management system built entirely on the Zoho platform: **Zoho CRM** as
the system of record for school staff, **Zoho Creator** as the parent-facing
application, connected with **Deluge**.

No separate backend, database, or frontend framework is used — the assignment
is implemented with native Zoho capabilities end to end.

## Why this stack

Zoho CRM already provides modules, relationships, validation rules, workflows,
roles/sharing, and reporting out of the box. Zoho Creator already provides a
lightweight app UI with its own security model and native CRM connectivity.
Building a custom backend/frontend on top would duplicate what the platform
gives for free, so this project deliberately stays inside CRM + Creator +
Deluge (see [docs/decisions.md](docs/decisions.md)).

## Repository layout

```text
docs/           architecture, data model, workflows, integration, security, reports, decisions
crm/
  modules/      field-level definitions for each CRM module
  workflows/    workflow rule specs (trigger -> action)
  functions/    Deluge function source (the reusable business logic)
  validations/  validation rule specs
  reports/      CRM report/dashboard specs
creator/
  pages/        Creator page specs (what the parent sees)
  forms/        Creator form specs
  reports/      Creator report specs (CRM data views)
  workflows/    Creator-side workflow specs
  functions/    Creator-side Deluge (mostly thin wrappers calling CRM)
  integrations/ CRM<->Creator connection config
  security/     parent access-control logic
```

Because Zoho CRM/Creator are configured through their web consoles rather than
a filesystem, everything under `crm/` and `creator/` is the **source-controlled
specification** of that configuration (Deluge scripts verbatim, module/field
definitions, workflow/validation specs) — it is what gets applied in the Zoho
org, kept in git for review and history instead of living only in the UI.

## Modules (Zoho CRM)

Standard: `Leads`

Custom: `Admissions`, `Students`, `Parents/Guardians`, `Academic Years`,
`Classes`, `Sections`, `Subjects`, `Teachers`, `Student Academic Enrollments`,
`Attendance`, `Examinations`, `Examination Results`, `Fees`, `Payments`.

Every module maps to a real requirement in the assignment — see
[docs/data-model.md](docs/data-model.md) for field-level detail and why each
one exists.

## Core flows

```text
CRM Webform -> Lead -> Admission -> Confirmed/Rejected -> Student

Student -> Student Academic Enrollment (per Academic Year) -> Class/Section

Student + Date            -> Attendance        (one record per day, enforced)
Student + Exam + Subject  -> Examination Result (marks validated)
Student -> Fee -> Payment(s) -> Collected / Outstanding / Status

CRM (source of truth) -> Creator (parent view, read-mostly, access-scoped)
```

## Documentation

- [docs/architecture.md](docs/architecture.md) — system design, layering, why-not-more
- [docs/data-model.md](docs/data-model.md) — entities, fields, relationships, history vs current state
- [docs/workflows.md](docs/workflows.md) — every automation, trigger to action
- [docs/integration.md](docs/integration.md) — CRM <-> Creator data flow
- [docs/security.md](docs/security.md) — parent-to-student access control
- [docs/reports.md](docs/reports.md) — management reports/dashboards
- [docs/decisions.md](docs/decisions.md) — architectural decisions and why

## Setup / configuration

This repo contains the specification to replicate in a Zoho org:

1. Create the CRM modules listed in `crm/modules/` with the fields/lookups described.
2. Create the Deluge functions in `crm/functions/` under Setup → Functions.
3. Wire the workflow rules in `crm/workflows/` to call those functions.
4. Add the validation rules in `crm/validations/`.
5. Build the Creator app pages in `creator/pages/` and connect them to CRM
   using the CRM integration task described in `creator/integrations/`.
6. Configure Creator user roles per `creator/security/`.

No credentials, API keys, or org-specific IDs are committed to this repo —
supply them via Zoho's own connection/environment configuration when applying
this spec to a real org.

## Known limitations

See [docs/decisions.md](docs/decisions.md) for the full list; briefly:

- This repo is a configuration specification, not a live Zoho org — applying
  it requires manual setup in Zoho CRM/Creator (no CLI/IaC tool covers 100%
  of CRM+Creator configuration).
- Notifications (email/SMS) are specified but require a configured
  Zoho Mail/SMS connection to actually deliver.

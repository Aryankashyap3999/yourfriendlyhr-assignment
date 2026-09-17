# Module: Leads (standard, extended)

**Purpose**: the admission enquiry, before any decision is made. Uses Zoho
CRM's standard Leads module plus a webform, rather than a custom module —
Leads already gives follow-up tracking, lead status, and conversion tooling
for free.

## Webform

`Admission Enquiry` webform, mapped fields below, posts directly into Leads.
Public URL is embedded on the school's admissions page; no custom code
needed — this is native CRM webform-to-lead capture.

## Added fields

| Field | Type | Required | Notes |
|---|---|---|---|
| Student Name (Proposed) | Text | Yes | the applicant, not yet a Student record |
| Class Applied For | Lookup → Classes | Yes | |
| Academic Year Applied For | Lookup → Academic Years | Yes | defaults to the current year on the webform |
| Enquiry Source | Picklist | Yes | Webform / Walk-in / Referral |

## Standard fields used

`Lead Status` (New / Contacted / Follow-up / Converted / Lost), plus the
standard Company/Phone/Email fields repurposed for parent contact details.

## Flow

```text
Webform submit → Lead (Status = New)
       ↓ (admissions staff, manual follow-up)
Status = Contacted / Follow-up
       ↓
Admission record created against this Lead (see admissions.md)
       ↓
Admission Decision = Confirmed → Lead Status = Converted
Admission Decision = Rejected  → Lead Status = Lost
```

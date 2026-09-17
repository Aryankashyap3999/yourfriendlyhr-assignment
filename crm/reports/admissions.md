# Reports: Admissions

All native CRM reports (standard report builder, grouped/filtered list
views) — no custom Deluge needed, this is exactly what CRM reporting is for.

| Report | Source | Grouping/Filter |
|---|---|---|
| Total Enquiries | Leads | count, by Enquiry Source |
| Confirmed Admissions | Admissions | filter `Decision = Confirmed` |
| Rejected Admissions | Admissions | filter `Decision = Rejected`, grouped by Rejection Reason |
| Follow-ups Pending | Leads | filter `Lead_Status in (Contacted, Follow-up)` |
| Conversion Rate | Leads | `count(Lead_Status = Converted) / count(all)`, dashboard KPI widget |

Dashboard: "Admissions Overview" combining the five above as widgets on one
CRM dashboard, filterable by Academic Year Applied For.

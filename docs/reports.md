# Reports & Dashboards

Full specs live under [crm/reports/](../crm/reports/), one file per domain.
This is the index management would actually look at.

| Domain | Reports | Detail |
|---|---|---|
| Admissions | enquiries, confirmed/rejected, follow-ups, conversion rate | [admissions.md](../crm/reports/admissions.md) |
| Students | total, by year/class/section | [students.md](../crm/reports/students.md) |
| Attendance | per-student %, low-attendance list, class-today, trend | [attendance.md](../crm/reports/attendance.md) |
| Examinations | student/class/subject performance, averages | [examinations.md](../crm/reports/examinations.md) |
| Fees | total, collected, outstanding, status breakdown | [fees.md](../crm/reports/fees.md) |

## Why plain CRM reports, not a BI tool

Every report above is a native CRM report/dashboard (group-by + aggregate,
filter, or a rollup/formula field feeding a standard report) — nothing here
needed Zoho Analytics or a custom chart component. The two computed fields
this required (`Attendance Percentage` on Student Academic Enrollments,
and the existing `Amount Collected`/`Outstanding Amount`/`Payment Status`
on Fees) already existed or were added specifically because a report needs
to filter/sort on them — see
[decisions.md #9](decisions.md#9-two-ways-to-get-attendance-percentage).

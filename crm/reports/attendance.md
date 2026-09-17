# Reports: Attendance

Built on the `Attendance Percentage` rollup/formula field added to Student
Academic Enrollments (see
[../modules/student-academic-enrollments.md](../modules/student-academic-enrollments.md)
and [docs/decisions.md #9](../../docs/decisions.md#9-two-ways-to-get-attendance-percentage))
so these are all native, filterable/sortable CRM reports — no Deluge.

| Report | Source | Grouping/Filter |
|---|---|---|
| Attendance Percentage (per student) | Student Academic Enrollments | current year, sorted by Attendance Percentage |
| Low-Attendance Students | Student Academic Enrollments | filter `Attendance Percentage < Attendance_Alert_Threshold` |
| Class Attendance (today) | Attendance | filter `Date = today`, grouped by Class + Status |
| Attendance Trend | Attendance | grouped by Date (weekly), stacked by Status, dashboard line chart |

Dashboard: "Attendance Overview" — Low-Attendance Students list, Class
Attendance today, and the trend chart, filterable by Class/Section.

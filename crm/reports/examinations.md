# Reports: Examinations

| Report | Source | Grouping/Filter |
|---|---|---|
| Student Performance | Examination Results | grouped by Student Academic Enrollment, per Examination — sum(Marks Obtained)/sum(Max Marks) as a report-level formula column |
| Class Performance | Examination Results | grouped by Class (via Student Academic Enrollment), per Examination |
| Subject Performance | Examination Results | grouped by Subject, per Examination |
| Average Marks | Examination Results | average(Marks Obtained) grouped by Subject or Class |

These are standard CRM summary reports (group-by + aggregate columns) —
no Deluge needed for the report itself. `calculateStudentPerformance`
(added in the examinations phase) is for the Creator dashboard's single-
student view, not for these bulk management reports, for the same reason
noted for attendance in [docs/decisions.md #9](../../docs/decisions.md#9-two-ways-to-get-attendance-percentage):
a report filtering/sorting many records needs a native aggregate, not a
per-record function call.

Dashboard: "Examination Overview" — Class Performance and Subject
Performance side by side, filterable by Examination.

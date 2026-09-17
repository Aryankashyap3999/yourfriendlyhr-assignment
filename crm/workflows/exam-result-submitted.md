# Workflow: Examination Result Submitted

**Module**: Examination Results
**Trigger**: before insert

## Steps

1. [../functions/validateExamMarks.deluge](../functions/validateExamMarks.deluge)
   — range, subject/class match, duplicate check.
2. Record created.
3. Performance figures are computed on demand via
   [../functions/calculateStudentPerformance.deluge](../functions/calculateStudentPerformance.deluge)
   — not stored, same reasoning as attendance percentage
   (see [../modules/attendance.md](../modules/attendance.md)).

## Error handling

A thrown validation error blocks the insert and is shown to whoever is
entering marks (teacher/office) with the specific reason, not a generic
failure.

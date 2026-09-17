# Workflows & Automation

Each workflow below follows the layering from [architecture.md](architecture.md):
Trigger → Validation → Business logic → CRM operation → Automation.

## 1. Admission

```text
CRM Webform submitted
       ↓
Lead created (Lead Status = New)
       ↓                                (manual, by admissions staff)
Follow-up / Contacted / Follow-up-again
       ↓
Admission record created, linked to Lead
       ↓
Decision = Confirmed ──────────────┐         Decision = Rejected
       ↓                            │                ↓
Workflow: "On Admission Confirmed"  │        Admission stays Rejected,
       ↓                            │        Lead Status -> Lost
generateStudentId()                 │
       ↓                            │
Student created                     │
       ↓                            │
Student Academic Enrollment created │
 (Academic Year = Applied For,      │
  Class = Applied For, Section =    │
  first section with capacity)      │
       ↓                            │
Lead Status -> Converted ◄──────────┘
```

Trigger: field update on `Admissions.Decision`.
Function: [[createStudentFromAdmission]] (calls `generateStudentId`, then
creates the Student + the first Student Academic Enrollment in one place, so
"confirming an admission always produces a consistent Student + enrollment
pair" is enforced in one function rather than trusted to whoever clicks
confirm).

## 2. Attendance

```text
Attendance form submitted (Student Academic Enrollment, Date, Status)
       ↓
validateAttendance()
   - Student Academic Enrollment must belong to the current Academic Year
   - Status must be one of Present/Absent/Half Day/Leave
   - reject if a record already exists for (Enrollment, Date)
       ↓ (pass)
Attendance record created
       ↓
calculateAttendancePercentage() re-run for that enrollment (on read, not
stored — see below)
       ↓
Below threshold? -> see workflow 5 (low attendance alert)
```

Duplicate prevention: enforced twice — a CRM validation rule on
`(Student Academic Enrollment, Date)` uniqueness (platform-level, always on)
*and* `validateAttendance` explicitly checks before insert so the failure
message is meaningful instead of a generic constraint error.

Attendance percentage is **not** stored on the enrollment record — storing a
derived value invites it going stale. It is computed on demand by
`calculateAttendancePercentage` (COQL count query) whenever a report or
Creator page needs it, and only cached at the report/dashboard layer if that
report already caches (Zoho Analytics/CRM dashboards do this natively).

## 3. Examination

```text
Marks submitted for (Examination, Student Academic Enrollment, Subject)
       ↓
validateExamMarks()
   - 0 <= Marks Obtained <= Max Marks
   - reject if a result already exists for the same
     (Examination, Enrollment, Subject)
   - Subject must belong to the Enrollment's Class
       ↓ (pass)
Examination Result created
       ↓
calculateStudentPerformance() available on demand for:
   - student total % for that Examination
   - class average for a Subject
   - subject-level average across a Class
```

Same duplicate-prevention pattern as attendance: a CRM validation rule plus
an explicit check in `validateExamMarks`.

## 4. Fees & Payments

```text
Payment record created (Fee, Amount, Date)
       ↓
validatePayment()
   - Amount > 0
   - Amount <= Fee.Outstanding Amount (no overpayment, unless a business
     override flag is explicitly set — off by default)
       ↓ (pass)
Payment saved
       ↓
calculateOutstandingFees()  -> Fee.Amount Collected = sum(Payments.Amount)
                               Fee.Outstanding Amount = Total Fee - Amount Collected
       ↓
updatePaymentStatus()       -> Paid / Partially Paid / Pending
```

`Amount Collected` and `Payment Status` are rollup/formula-backed where CRM's
native rollup fields can do it; `calculateOutstandingFees` /
`updatePaymentStatus` exist as explicit Deluge only where the platform's
native rollup falls short of the exact rule (e.g. status has three states,
not a simple sum), keeping the "computed, never manual" guarantee either way.

## 5. Low attendance alert (additional feature)

See [decisions.md](decisions.md) for the full write-up of why this was
chosen. Summary:

```text
Attendance record saved
       ↓
calculateAttendancePercentage() for that student, current academic year
       ↓
Percentage < Attendance Alert Threshold (configurable, see below)?
       ↓ yes
Already alerted today for this student? -> skip (no repeat spam)
       ↓ no
getParentStudent()  -> resolve the linked Parent/Guardian
       ↓
sendLowAttendanceAlert()  -> email via Zoho Mail send-mail (Deluge)
```

The threshold is a single custom setting (`Attendance Alert Threshold %`,
default 75) read by `calculateAttendancePercentage`'s caller — changing school
policy is a one-field edit, not a code change.

## Deluge function catalog

| Function | Responsibility (single) | Called from |
|---|---|---|
| `generateStudentId` | produce the next unique `STU-YYYY-00001` | `createStudentFromAdmission` |
| `createStudentFromAdmission` | create Student + first enrollment on confirmation | Admission "Decision = Confirmed" workflow |
| `validateAttendance` | reject invalid/duplicate attendance before insert | Attendance form workflow |
| `calculateAttendancePercentage` | present/(present+absent+half-day-weighted) for a student+year | Reports, Creator, low-attendance alert |
| `validateExamMarks` | reject invalid/duplicate marks before insert | Examination Result form workflow |
| `calculateStudentPerformance` | student/class/subject aggregates for an exam | Reports, Creator |
| `validatePayment` | reject non-positive or overpaying payments | Payment form workflow |
| `calculateOutstandingFees` | recompute Amount Collected / Outstanding on a Fee | Payment created/edited workflow |
| `updatePaymentStatus` | set Fee.Payment Status from collected vs total | same, right after `calculateOutstandingFees` |
| `getParentStudent` | resolve Parent -> their Student(s), and the reverse check used by security | Creator pages, low-attendance alert, [[security]] |
| `sendLowAttendanceAlert` | compose and send the alert email | Low attendance alert workflow |

# Decisions

Short ADR-style log of choices that weren't obvious from the code/spec alone.

## 1. Zoho CRM + Creator only, no custom backend/frontend

**Context**: the assignment allows a traditional stack if genuinely needed.
**Decision**: stay entirely inside CRM + Creator + Deluge.
**Why**: nothing in the requirements (webform intake, academic records,
attendance, exams, fees, parent portal) needs anything CRM/Creator can't do
natively. Introducing a separate stack would mean building and syncing a
second copy of school data for no functional gain.

## 2. Student Academic Enrollment as a separate module, not fields on Student

**Decision**: class/section/year live on `Student Academic Enrollments`
(one row per student per year), not on `Students`.
**Why**: overwriting a `current_class` field every year destroys history.
A separate per-year row is the simplest structure that keeps every year's
placement intact and makes "current" a simple filter (`Academic Year.Is
Current = true`) rather than a snapshot that has to be maintained by hand.

## 3. Admissions kept separate from Leads

**Decision**: `Admissions` is its own module linked to `Leads`, rather than
extra fields bolted onto `Leads`.
**Why**: Lead nurturing (follow-ups, contact status) and the admission
decision (confirm/reject, which class/year) are different responsibilities
with different lifecycles. Mixing them into one module makes both harder to
report on cleanly.

## 4. Duplicate prevention done at two levels

**Decision**: attendance and exam-result uniqueness are enforced by both a
CRM validation rule and an explicit check inside the Deluge function.
**Why**: the validation rule is the actual guarantee (it can't be bypassed by
a different entry point); the Deluge check exists purely to produce a
readable error message instead of a generic constraint failure. This is
belt-and-suspenders on purpose, not redundant — removing either changes
behavior (drop the rule and a non-Deluge entry point could create a
duplicate; drop the Deluge check and the error message gets worse).

## 5. Fee amounts are always derived, never hand-entered

**Decision**: `Amount Collected`, `Outstanding Amount`, and `Payment Status`
on `Fees` are all computed from `Payments`, never editable by staff.
**Why**: the assignment explicitly calls out that these should come from
business logic rather than manual input — manual entry is exactly the kind
of thing that drifts out of sync with reality.

## 6. Additional feature: low attendance alert

**Problem**: attendance is tracked, but nobody proactively notices a student
sliding below an acceptable attendance level until report card time.
**Why it matters**: catching it early is the whole point of tracking
attendance daily instead of termly.
**Business rule**: if a student's attendance percentage for the current
academic year drops below a configurable threshold (default 75%) after a new
attendance record is saved, and no alert has already gone out today for that
student, notify the linked parent by email.
**Implementation**: `calculateAttendancePercentage` + `getParentStudent` +
`sendLowAttendanceAlert`, chained from the attendance-save workflow — no new
module, no polling job, just one more step after attendance is validated.
**Scalability**: triggered per attendance record (event-driven), not a nightly
scan over every student — cost stays proportional to attendance actually
taken that day, not total student count.

## 7. No caching, no custom auth, no sync job

**Decision**: explicitly not built — see [architecture.md](architecture.md#what-was-deliberately-not-built).
**Why**: each would solve a problem this system doesn't have at its expected
scale (a single school). Adding them now would be optimizing before there's
a measured need.

# 0009 — Conflicts are surfaced, never auto-resolved

2026-08-27 · Accepted

## Context

A nutritionist sets a 2100 kcal floor on 14 Aug. A goal set on 1 Aug implies ~1900 kcal. Both
are stored, both have dates, and nothing compares them — so the weekly advisor reads whichever
it happens to read and either overrides a professional or quietly misses the target, depending
on prompt luck. Nobody is told.

## Decision

1. **Compare by date.** `goals.horizon.starts_on` against `assessments.taken_on`. Recency tells
   you which conversation to have, never who is right — a goal set *after* a constraint means
   the user may already have chosen.
2. **Never auto-resolve.** Surface it.
3. **Resolution edits the plan.** The outcome is a new goal, routine or diet version whose
   `created_from.reason` names the conflict — so it cannot be raised again next week.
4. **Until resolved, the constraint holds.** A hard constraint gates recommendations while the
   question is open. The advisor does not suggest what a doctor forbade.

## Consequences

No schema change was needed — both sides already carry dates. What was missing was `proposals`
(ADR-0005) to hold the open question.

`assessments.source` distinguishes `user_reported` from `document`: "he told me 2100" is weaker
evidence than an uploaded InBody report, and the system should know the difference.

Active hard constraints are injected into every agent prompt above goals and above the routine
(`SV-8`). That is a safety property, not a nicety.

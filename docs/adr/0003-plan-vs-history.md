# 0003 — Plan and history are separate zones

2026-08-27 · Accepted

## Context

Early sketches mixed intent and record: the training split was a hardcoded constant in code and
nutrition targets sat on the user profile. Nothing in that shape could answer the one question
the product exists for — "did you do what you meant to?" 

## Decision

Every domain splits in two, and the join between them is the product:

| Domain | Plan | History | Join |
|---|---|---|---|
| Training | `routines` | `sessions` | adherence, substitutions |
| Food | `goals.measurables` | `meals` | gap |
| Body | `goals.measurables` | `daily_metrics` | trend |

`goals` sits above all three and gives them direction.

## Consequences

Adherence, deviation and "am I on track" become queries instead of prompt work. It also means
the plan must be stored as data, versioned and immutable — a routine is never edited in place,
it is superseded.

The cost is more collections. Mitigated by the zone grouping: five words, not fifteen names.

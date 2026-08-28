# 0004 — The routine is the vocabulary

2026-08-27 · Accepted

## Context

The obvious approach is to canonicalize free text into an exercise name. That works for one user
with one notation and breaks on the second — every user writes differently (`10 - 60lb` versus
`3x10 @ 60`), and open-ended canonicalization is neither accurate nor gradeable.

## Decision

Users define a routine. Every logged message is resolved against **that user's ~30 exercises**,
not against open vocabulary. Notation stops mattering; matching is the whole job.

Resolution is a ladder, cheapest first: normalize → alias index (Redis) → this routine day's
candidates → everything the user has used → model against a closed list → ask.

## Consequences

Three things that would otherwise need an LLM to infer became database queries: substitution
detection, volume-vs-plan, and "is this the same movement". Matching against a closed list is
also trivially gradeable in an eval, which open canonicalization never was.

Every answered question widens the free top step — the flywheel. Aliases are capped at 12 per
exercise and evicted by lowest hit count, because a bad alias is permanently wrong.

Requires a routine to exist, which makes onboarding (`ON-1`) the real gate on the product.

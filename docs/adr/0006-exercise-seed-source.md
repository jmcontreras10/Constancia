# 0006 — free-exercise-db as the seed, not RepDB

2026-08-27 · Accepted

## Context

Two candidate datasets were reviewed.

**RepDB** (`RepDB/exercise-dataset`) — 250 exercises, native Spanish names matching the owner's
own phrasing, MET values, difficulty, instructions. Licence is custom: MIT for the viewer code,
proprietary for the data. Term 3 forbids republishing the dataset in a repository.

**free-exercise-db** (`yuhonas/free-exercise-db`) — 873 exercises, **Unlicense (public domain)**,
English only, thinner metadata.

## Decision

free-exercise-db, vendored into the repo. A one-time Spanish name pass (`ON-8`) produces
`name.es`, which is ours to keep because the source is public domain.

## Consequences

Constancia is MIT and self-hosted, so committing RepDB's JSON would be republishing it in a dataset
repository — a breach, not a grey area. The workaround (fetch at setup, never vendor) works, but
the attribution and no-redistribution obligations then ride along to **every self-hoster,
forever**.

The Spanish argument that initially favoured RepDB was over-weighted: alias learning means the
library's language matters at only two moments — browsing while building a routine, and the very
first time a name is written. After that the user's own phrasing is stored. It is a two-week
problem, not a permanent one.

Activities neither dataset covers — walking, football, meditation — are hand-authored (`ON-9`)
with `seed_source: "constancia"`, in the same collection under `category`.

RepDB remains a good fallback if the translated names disappoint, or if MET values and images
become worth the licence.

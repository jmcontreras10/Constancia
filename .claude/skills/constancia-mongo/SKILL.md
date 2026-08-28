---
name: constancia-mongo
description: Constancia data-layer conventions — user scoping, dates, indexes, versioning. Use when touching src/db/, defining a collection, or writing any query.
---

# Data layer

Shape of every collection: `docs/schema/collections.json`. That file is the source of truth; the
Miro board is a rendering of it.

## Non-negotiable

**Every query filters by `user_id`** (INV-2). This is multi-tenant health data — one missed
scope leaks another person's medical records. The only escape hatch is a commented reason:

```ts
// inv-2: ok — exercise_library scope:"global" rows are shared by design
```

## Dates

- `date_key: string` as `"YYYY-MM-DD"`, the **user-local** calendar day. MongoDB cannot store a
  bare date, and the local day is a decision made at write time — a 1 a.m. session belongs to
  yesterday's training day.
- `day_key` is a different field: which *routine* day (`"Pull"`). Sessions carry both.
- All day arithmetic lives in `src/core/time.ts` (INV-7).

## Versioning

`routines` and `goals` are **immutable**. Editing creates a new document with `supersedes`
pointing back, `ends_on` set on the old one, and `created_from.reason` explaining why. History
keeps pointing at the version that was true at the time.

## Copy, don't reference

A `meal` stores its own macros even when it came from a template. A `session` stores its own
load values. Correcting a template or a routine must never rewrite months of logged history.

## Indexes

Declared in `ensureIndexes()`, called at startup, never ad hoc. Two TTLs doing different jobs:
`messages.received_at` at 90 days, `entry_drafts.expires_at` at ~12 hours. Dedupe keys live in
Redis at 7 days, not in Mongo.

`daily_summaries` has a unique `(user_id, date_key)` and is written by **upsert** — the nightly
recompute and the write path both call the same helper.

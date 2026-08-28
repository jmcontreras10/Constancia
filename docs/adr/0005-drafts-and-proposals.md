# 0005 — Two conversation states, not zero

2026-08-27 · Accepted

## Context

A single message is often not a complete record — half a workout, a meal with no quantity. And
when the system wants to suggest a plan change, a WhatsApp message scrolls away: the question is
either forgotten or asked again next week.

## Decision

Two collections, mirroring each other:

- **`entry_drafts`** — incoming. A partial entity plus `missing[]` and `unresolved[]`. Nothing
  reaches a domain collection except through a complete draft.
- **`proposals`** — outgoing. A suggested change to a goal, routine or diet, with `status`,
  `expires_at` and `outcome_ids`.

## Consequences

**The model proposes; code decides.** Which fields are required comes from the Zod schema and
the routine's `required` flags — never from the model's opinion about whether it has enough.
The agent fills gaps; it never rules that a gap is acceptable.

Code also picks *which* gap to ask about and when the draft is complete. The model only phrases
the question. A hard budget of ~2 questions means it commits with `incomplete: true` rather than
interrogating the user into not logging at all.

`proposals` additionally supports `partially_accepted` — a knee constraint may change both the
routine and the goal, and the user may want only one.

`confidence` was deliberately dropped from drafts: a single float invites an arbitrary threshold
and hides *what* was uncertain. `missing` and `unresolved` are already decision-ready.

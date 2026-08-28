# Specs

One file per slice. `/slice N` reads `NN-*.md` and builds it. You never paste a design document
into a prompt.

**Not yet written.** The data model settled first, deliberately — it is what every slice depends
on. These are outlines of what each slice must cover, to be filled in at step 1 of the build
order.

| # | Slice | Must cover | Depends on |
|---|---|---|---|
| 01 | Repo scaffold | pnpm workspace, `services/api` + `services/dashboard` + `packages/contracts`, Docker Compose, `/health`, CI lint+types+tests | — |
| 02 | Data layer | Zod schemas and Mongo collections from `schema/collections.json`, `ensureIndexes()` at startup, `date_key` and `src/core/time.ts` | 01 |
| 03 | Intake pipeline | Twilio webhook + signature validation, ack under 200 ms, dedupe on `provider_message_id` via Redis, triage gate, `messages` with TTL | 02 |
| 04 | Audio agent | Hosted STT behind an adapter, transcript onto `messages.media[]`, language pinned from profile, cost logged with `provider: "stt"` | 03 |
| 05 | AI gateway | `ModelRouter`, `TASK_ROUTES`, tier selection, structured outputs, cost logging to `model_calls`, per-user spend cap | 02 |
| 06 | Resolution | The ladder: normalize → Redis alias index → routine-day candidates → user's exercises → model → ask. Alias cap and eviction. Meal template matching | 05 |
| 07 | Drafts and proposals | `missing`/`unresolved`, one question at a time chosen by code, question budget, commit path, `proposals` lifecycle | 06 |
| 08 | Auth and identity | Web JWT, WhatsApp link code, phone as the chat credential, account operations web-only | 02 |
| 09 | Jobs | Scheduler in its **own container** — not the web process, which runs multiple workers — daily check-in, weekly review, `daily_summaries` recompute | 05 |
| 10 | Product agents | Exercise, Nutrition, Orchestrator. Hard constraints injected above goals. Rubrics alongside | 05, 09 |
| 11 | Dashboard | Next.js + MUI, types from `packages/contracts`, charts branching on `load.better`, loading and error states | 01 |
| 12 | DevOps | Dockerfiles, Compose prod overrides, nginx, GHCR, deploy | 01 |

## Writing a spec

A spec is an instruction to a builder, not prose about a decision. It states what exists when
the slice is done, and the specific traps in that area. Reasons and rejected options belong in
an ADR.

Every spec ends with a **definition of done** that includes: tests, at least one eval case if it
touches AI, `check-invariants` passing, and contracts regenerated if the HTTP surface changed.

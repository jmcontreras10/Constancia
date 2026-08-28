# Constancia backlog

Priority: **1** = active now · **2** = next up · **3** = after the core works
Status: `doing` · `todo` · `blocked` · `done`

Update the status column as things move. One row per thing worth tracking separately.

---

## Data layer

| ID | P | Status | Item | Notes |
|---|---|---|---|---|
| DL-1 | 1 | done | Exercise library + per-user overlay | Aliases capped at 12, LFU eviction, Redis index. Seed source still open (Q-3) |
| DL-2 | 1 | done | Routines | User-named days, no load targets, date-versioned with `supersedes` |
| DL-3 | 1 | done | `workout_sessions` — strength + activity | Two shapes: sets/reps, and duration/distance for anything else |
| DL-4 | 1 | done | `goals` | Narrative + measurables. Free text and reference images are the source of truth |
| DL-5 | 1 | done | Intake: `messages` + `entry_drafts` | Draft is the conversation state |
| DL-6 | 1 | done | Indexes + retention policy | Mongo TTL 90d on messages; Redis 7d on dedupe keys |
| DL-7 | 1 | done | `meals` | No per-ingredient logging. Macros copied onto the meal, never read through the template |
| DL-13 | 1 | done | `proposals` | Outgoing plan-change suggestions. Mirrors `entry_drafts` for incoming data |
| DL-14 | 1 | done | Bilingual naming | English canonical, Spanish secondary. `name: {en, es}`, per-user display preference |
| DL-15 | 1 | done | Activities in `exercise_library` | Walk, football, meditation get the same catalog and `category`, not a free string |
| DL-12 | 1 | done | `meal_templates` | Emerge from repeats or named outright. Per-user only — nobody sees anyone else's food |
| DL-8 | 1 | done | `daily_metrics` | Weight, steps, water, sleep in one collection. `aggregation` decides sum vs. point |
| DL-10 | 1 | done | `sessions` — strength / cardio / recovery | Renamed from `workout_sessions`. Holds meditation and hiking too |
| DL-11 | 1 | done | `assessments` | Medical & professional input. Hard constraints outrank goals in agent prompts |
| DL-9 | 2 | todo | Derived: `daily_summaries`, `agent_reports` | Recomputable; safe to drop and rebuild |

## Services

| ID | P | Status | Item | Notes |
|---|---|---|---|---|
| SV-1 | 2 | todo | Auth: web JWT + WhatsApp linking | Phone is the credential; no session on the chat side. Deferred by owner |
| SV-2 | 2 | todo | Twilio webhook: signature validation + idempotency | Security hole in the draft doc. Not optional |
| SV-3 | 2 | todo | Message triage gate | Heuristics first, cheap model second. Irrelevant messages never persisted |
| SV-4 | 2 | todo | Draft question loop | Code picks the question, model phrases it, hard budget of ~2 |
| SV-5 | 2 | todo | AI gateway + task routing | Native `@anthropic-ai/sdk` default; adapter pattern for others. ADR-0010 |
| SV-6 | 3 | todo | Per-user daily spend cap | Reads `model_calls`. Needed before any public deploy |
| SV-8 | 2 | todo | Hard-constraint injection into agent prompts | Active `assessments.constraints` rank above goals and routine. Safety, not a nicety |
| SV-11 | 3 | todo | Conflict detection: goal vs. hard constraint | Compare dates, surface to the user, never auto-resolve. Resolution creates a new plan version |
| SV-12 | 2 | todo | Nutrition sub-agent | `task_name: estimate_meal_macros`. Reads labels and recipes at template time; estimates plates only as fallback |
| SV-13 | 2 | todo | Meal template resolution ladder | exact name → recent/frequent → fuzzy → estimate. Same shape as exercise matching |
| SV-14 | 3 | todo | Image-hash cache for vision calls | A resent or Twilio-retried photo must never re-bill |
| SV-15 | 3 | todo | Spend cap degrades to Batch API | Over cap, queue for the nightly batch at half price. Never refuse |
| SV-16 | 1 | todo | Audio agent | Dedicated component, hosted STT API + own key. Transcribes and stops — never interprets. Behind a swappable adapter like the LLM gateway |
| SV-17 | 2 | todo | Echo quantities heard, not just results | Speech-to-text fails on numbers, and numbers are the whole payload. Confirmations must restate the grams |
| SV-18 | 2 | todo | Pin transcription language from profile | Auto-detection breaks on mixed Spanish/English, which is how the owner actually talks |
| SV-19 | 2 | todo | Log STT calls to `model_calls` | `provider: "stt"`, `audio_seconds`. One ledger so the spend cap covers transcription too |
| SV-9 | 3 | todo | Private media storage + presigned URLs | Covers goal reference images and assessment PDFs. Health data, not workout logs |
| SV-10 | 3 | todo | Account + data deletion path | Stops being optional once clinical records are stored |
| SV-7 | 3 | todo | Scheduler in its own container | Must not run in the web process — multiple workers fire jobs twice |

## Onboarding

| ID | P | Status | Item | Notes |
|---|---|---|---|---|
| ON-1 | 2 | todo | Routine creation flow | The hardest UX in the product. Everything downstream depends on it existing |
| ON-2 | 2 | todo | Exercise library seeding + search | User picks from the library while building a routine |
| ON-5 | 2 | todo | `scripts/seed_exercises.py` | If free-exercise-db: vendored, loader only. If RepDB: **must fetch at setup, never vendor** — licence term 3 forbids redistributing it in a repo |
| ON-6 | 3 | todo | RepDB attribution in the app UI | **Only if Q-3 lands on RepDB.** Term 2 needs a visible link in about/credits, not just the README, or every self-hoster silently breaches it |
| ON-7 | 3 | todo | Ask RepDB about the "API" wording | **Only if Q-3 lands on RepDB.** support@repdb.co — does Constancia's own `/exercises` endpoint count as in-app use? |
| ON-8 | 2 | todo | Spanish name pass over free-exercise-db | 873 names, LLM-translated then reviewed. Ours to vendor — source is public domain. Feeds `name.es` and `aliases_global.es` |
| ON-9 | 2 | todo | Hand-author the activity catalog | ~30 entries neither dataset has: walk, football, hike, swim, yoga, meditation, sauna. `seed_source: "constancia"` |
| ON-3 | 2 | todo | Goal setting | Feeds `goals`; needed before agents can say anything useful |
| ON-4 | 3 | todo | WhatsApp link UX | Code generation, expiry, re-link |

## Docs & decisions

| ID | P | Status | Item | Notes |
|---|---|---|---|---|
| DOC-1 | 1 | done | `CLAUDE.md` | Rewritten for TypeScript + MUI. The always-loaded contract |
| DOC-2 | 1 | done | `docs/ARCHITECTURE.md` | The build harness: six layers, four agents, five skills, four pipelines |
| DOC-3 | 1 | done | `docs/harness/invariants.md` + `evals.md` | The eight rules and the eval design |
| DOC-4 | 1 | done | `docs/adr/` — 10 records | Every decision from this design phase, with what was rejected |
| DOC-5 | 1 | done | `.claude/skills/` — 5 skills | slice · gateway-task · agent-prompt · eval · mongo |
| DOC-6 | 2 | todo | Fill `docs/specs/01..12` | Outlines exist in `specs/README.md`. Step 1 of the build order |
| DOC-7 | 3 | todo | Root `README.md` | Public-facing. Written last, when there is something to show |

## Dev harness

| ID | P | Status | Item | Notes |
|---|---|---|---|---|
| HR-1 | 1 | todo | `scripts/check-invariants.ts` + hooks | The 8 rules in `docs/harness/invariants.md`, enforced not hoped for. Hooks stay off until the script exists |
| HR-2 | 2 | todo | Parser golden set | 40–60 cases from owner's real WhatsApp history. Only he can write these |
| HR-3 | 3 | todo | Synthetic fixtures with planted signals | Makes the cross-domain correlation claim testable |
| HR-4 | 3 | todo | CI gates | lint + invariants + tests per PR; eval-lite on AI diffs |
| HR-5 | 3 | todo | `/slice`, `/eval`, `/promptdiff` | Pipelines. `/promptdiff` first — prompt edits have no diff signal |
| HR-6 | 2 | todo | Generator: `collections.json` → Zod + Miro | One source of truth. Stops the schema, the contracts and the board drifting apart |
| HR-7 | 3 | todo | CI check: `collections.json` matches the Zod schemas | The drift guard. Without it HR-6 rots in a month |

## Product

| ID | P | Status | Item | Notes |
|---|---|---|---|---|
| PR-1 | 2 | todo | Chat surface vs. dashboard scope | Owner leaning chat-first. Data model unaffected either way |
| PR-2 | 3 | todo | Charts | Recharts. Weight trend, strength progression (must branch on `load.better`), adherence, weighed-vs-estimated meals |
| PR-3 | 3 | todo | Weekly review agents | Exercise + Nutrition + Orchestrator |
| PR-4 | 3 | todo | Health integration for steps | Typing a step count daily is the exact friction Constancia exists to remove. Apple Health / Google Fit |

---

## Open questions

| ID | Status | Question | Waiting on |
|---|---|---|---|
| Q-1 | answered | `date_key` vs `day_key` → **both kept.** User-local day is a write-time decision, not a query-time derivation | — |
| Q-2 | answered | Day naming → **stable `key`, editable `label`.** Trainer agent names from content; `Day 1` is the fallback | — |
| Q-3 | answered | Seed source → **free-exercise-db, vendored.** English-primary settles it: its English names become canonical, Spanish is a layer we add and own | — |
| Q-7 | answered | How to model goals? → **narrative + reference images are truth; measurables are a revisable interpretation** | — |
| Q-8 | answered | Walks, meditation, water — where? → **time set aside = `sessions`; a number that accumulates = `daily_metrics`** | — |
| Q-9 | answered | Separate `goal_revisions`? → **No.** `supersedes` chain is the history; `created_from` is the why; `assessments` holds the evidence | — |
| Q-11 | answered | Transcription backend → **hosted API with its own key**, behind a swappable adapter. Self-hosting stays possible later, not a launch blocker | — |
| Q-10 | answered | Constraint vs. goal → **compare by date, never auto-resolve, surface it; resolution creates a new plan version. Constraint holds while open** | — |
| Q-4 | answered | Routine day partly free? → **Yes. User records anything; adherence measures, never blocks** | — |
| Q-5 | answered | Training outside any routine? → **Recorded as an activity session, still counted toward goals** | — |
| Q-6 | answered | Confidence score on drafts? → **Dropped. Add back only if early adopters need it** | — |

# Constancia

AI fitness tracker at **constancia.fit**. Users log workouts, meals and body metrics as **WhatsApp messages** —
increasingly **voice notes** — and an AI pipeline parses, stores and coaches on them.
A web dashboard shows charts and hosts a chat. MIT, self-hostable.

**Status: design complete, no application code yet.** The data model is settled
(`docs/schema/collections.json`, 15 collections). Build order lives in `docs/BACKLOG.md`.
Do not scaffold ahead of a spec.

---

## Stack

**TypeScript everywhere.** See `docs/adr/0001-typescript-over-python.md`.

- `services/api/` — Fastify + Zod + MongoDB driver + BullMQ. All business logic.
- `services/dashboard/` — Next.js App Router + **MUI** + Recharts.
- `packages/contracts/` — Zod schemas shared by both. The HTTP boundary is typed at both ends.
- Channel: **WhatsApp via Twilio**. Telegram is a future adapter — do not build it now.
- Everything runs in Docker Compose. No Vercel, no Kubernetes.

---

## Hard rules

Architecture, not preference. Breaking one is a bug even if the code runs.

1. **Only `src/gateway/` imports a model SDK.** Everything else calls `router.complete()`.
2. **Every MongoDB query filters by `user_id`.** No exceptions. Multi-tenant health data.
3. **Every AI call passes a `taskName` registered in `TASK_ROUTES`.** No ad-hoc model strings.
4. **`/webhook/*` validates the Twilio signature.** Without it anyone can write to any account.
5. **The two services never import each other.** The contract is `packages/contracts`.
6. **Prompts are module constants.** Never interpolate user text into a prompt — WhatsApp
   input is untrusted.
7. **No day arithmetic outside `src/core/time.ts`.** The server is UTC; users are not.
8. **No secrets in the repo.** Never write `.env`, `infra/certs/`, or any `*.pem`.

Enforced by `scripts/check-invariants.ts` on every edit and in CI. *(Not built — HR-1.)*

---

## Domain conventions

Full typed schema: `docs/schema/collections.json`. Visual: the Miro board linked there.

**Names are English-canonical, Spanish-secondary.** `name: { en, es }`. Users can display
both. Matching runs against both plus learned aliases.

**Load is not always weight.** Every exercise carries `default_load.type` and
`better: "higher" | "lower"`. An assisted pull-up machine reads 1–18 and **lower is stronger** —
any trend calculation that ignores this reports regression as progress.

**Store raw and normalized.** `load: { value, unit }` as the user said it; `load_kg` for
analytics. Confirmations echo the user's own numbers back.

**Two date fields, different jobs.** `date_key` is the user-local calendar day
(`"2026-08-26"`). `day_key` is which routine day (`"Pull"`). Sessions carry both.

**The routine is a reference, never a constraint.** Users log whatever they did. Adherence is
measured afterwards; nothing is rejected for being off-plan. Deviation is the advisor's raw
material, not an error.

**Copy, don't reference, for history.** A meal stores its own macros even when it came from a
template. Correcting a template must never rewrite months of logged history. Same for
`routine_ref` against immutable routines.

**Strip invisible Unicode before parsing.** WhatsApp copy-paste inserts word-joiners (U+2060)
and zero-width spaces mid-line.

---

## Repo map

```
services/api/          Fastify. src/{agents,channels,gateway,jobs,resolve,routes,db,core}
services/dashboard/    Next.js. src/{app,components,lib}
packages/contracts/    Zod schemas shared across the HTTP boundary
docs/schema/           collections.json — source of truth for the data model
docs/specs/            Build plan, one file per slice
docs/adr/              Decision records
docs/harness/          Invariants and eval design
evals/                 Golden cases, rubrics, baseline.json
scripts/               check-invariants.ts, seed.ts
```

---

## How to work here

- **Read the spec file first.** `docs/specs/NN-*.md` is the instruction, not a suggestion.
- **One slice at a time.** Finish it — code, tests, eval case, docs — before starting another.
- **Deviating from a spec is allowed; doing it silently is not.** Write an ADR.
- **Prompt changes need `/promptdiff`.** A prompt edit passes every test and can still make
  output worse. It is the only change here with no signal in the diff.
- Spanish in user-facing strings and fixtures. English in code, comments and docs.

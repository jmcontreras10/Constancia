# The build harness

How Constancia gets built, not what it does. For what it does, read
[`schema/collections.json`](schema/collections.json).

The goal is not to use as many agents as possible. It is to **make it safe to let an agent write
code fast** — which means the things that must not break are checked by machines, not by prose.

*Deterministic guardrails at the bottom, judgment at the top, and exactly one seam worth
parallelizing.*

---

## 1. Six layers

```
  L0  GROUND TRUTH  (no AI)
      CLAUDE.md · docs/specs/NN-*.md · docs/adr/ · docs/schema/collections.json
      The plan lives in the repo. It is never a paste buffer.
                              │  read by everything below
  ────────────────────────────┼───────────────────────────────────────────────
  L1  GUARDRAILS  (deterministic, zero tokens)
      hooks →  biome · tsc --noEmit · vitest · check-invariants.ts
      PreToolUse   deny writes to .env, certs, *.pem
      PostToolUse  format + invariant-check the file just touched
                              │  constrains
  ────────────────────────────┼───────────────────────────────────────────────
  L2  SKILLS  (procedural knowledge, loaded on demand)
      slice · agent-prompt · gateway-task · eval-case · mongo
                              │  loaded by
  ────────────────────────────┼───────────────────────────────────────────────
  L3  AGENTS  (four, narrow tools, narrow paths)
      spec-keeper      api-builder   ║   dash-builder      eval-runner
      (docs/ only)     (services/api)║   (services/dash)   (read-only)
                                     ↑
                        packages/contracts = the seam
                              │  orchestrated by
  ────────────────────────────┼───────────────────────────────────────────────
  L4  PIPELINES   /slice N   /eval   /promptdiff   /ship
                              │  enforced again in
  ────────────────────────────┼───────────────────────────────────────────────
  L5  CI
      every PR:   biome + tsc + invariants + vitest        (0 tokens, seconds)
      AI diffs:   eval-lite, 10 golden cases, cheap tier   (~$0.01)
      nightly:    eval-full + rubric judges + cost report  (~$0.30)
```

---

## 2. Five principles

**1. If a machine can check it, a machine checks it.**
See [`harness/invariants.md`](harness/invariants.md). Eight rules, ~20 lines of checker each.
This is the highest-leverage thing in the repo.

**2. One seam, two builders.**
The only hard boundary is that the two services never import each other. That is the one place
parallel agents don't produce merge garbage. Everything else — gateway, resolver, jobs, agents —
is tightly coupled and belongs in one sequential context. Resist spawning six agents.

**3. The spec is a repo artifact.**
`/slice N` reads `docs/specs/NN-*.md`. You never paste a design doc into a prompt. When code
deviates, `spec-keeper` amends the spec and writes an ADR — so the docs cannot silently rot,
which is the failure mode of an append-only amendments file.

**4. Prompts are code, so prompts need tests.**
A prompt edit is the highest-risk change here and the only one with no signal in the diff: the
code compiles, the tests pass, the output quietly gets worse. `/promptdiff` runs the affected
component's eval on `HEAD` and on the working tree and prints the delta. Build it early.

**5. Cheap model in the inner loop, strongest model as the judge.**
Golden cases run against the tier the task will actually use in production. An LLM-as-judge
always runs on the strongest available model — a weak judge is worse than no judge.

---

## 3. The seam

`packages/contracts` holds Zod schemas for every request and response. The API validates against
them; the dashboard infers its types from them. Because both sides are TypeScript, the contract
is a **compile error**, not a runtime surprise — which is most of why the stack decision in
[ADR-0001](adr/0001-typescript-over-python.md) was worth making.

`api-builder` owns the schemas. `dash-builder` imports them and may not edit them; if the
dashboard needs a field that does not exist, that is reported, not added.

---

## 4. Agents

Four. Each has narrowed tools and a narrowed path scope. Definitions live in `.claude/agents/`.

| Agent | Writes to | Job |
|---|---|---|
| `api-builder` | `services/api/**`, `packages/contracts/**` | Backend: db, gateway, resolver, jobs, routes, tests |
| `dash-builder` | `services/dashboard/**` | Next.js + MUI + Recharts. Reads contracts, never backend source |
| `spec-keeper` | `docs/**` only | Reconciles spec against built code; writes ADRs. Cannot "fix" code |
| `eval-runner` | nothing | Runs suites, reports deltas. Read-only so it cannot loosen a failing test |

The read-only and docs-only scopes are the point, not a limitation. An agent that grades must
not be able to change what it grades.

Built-ins cover the rest: `Explore` for search, and the `code-review` / `security-review`
skills. No need to rebuild those.

---

## 5. Skills

Skills are *how we do X in this repo*. They carry the house pattern so it is not re-derived —
or re-invented differently — in every context. Definitions in `.claude/skills/`.

| Skill | Triggers on | Carries |
|---|---|---|
| `constancia-slice` | "implement spec N", "add a feature" | The order that makes a change complete: spec → contract → model → index → gateway task → route → test → eval case → ADR |
| `constancia-agent-prompt` | editing `src/agents/**`, `src/resolve/**` | House style for product-agent prompts, the required rubric, the mandatory `/promptdiff` |
| `constancia-gateway-task` | adding an AI or STT call | Register `taskName` → pick tier → note calls/day → add eval case → confirm cost logging |
| `constancia-eval` | writing or running evals | Golden-set format, harvesting a production failure into a case, reading the table |
| `constancia-mongo` | touching `src/db/**` | `user_id` scoping, `date_key` convention, index-on-startup, the `daily_summaries` upsert |

The split from agents matters: **skills are knowledge, agents are isolation.** Most of what
people reach for a subagent to do is really a skill.

---

## 6. Pipelines

| Command | Does |
|---|---|
| `/slice <n>` | Read spec → plan → implement → `vitest` → `/eval` affected suites → `spec-keeper` → report deviations |
| `/eval [suite]` | Run golden set, print per-field accuracy against `evals/baseline.json` |
| `/promptdiff` | Detect changed prompt constants, run the suite before and after, print the delta |
| `/ship` | Invariants + tests + eval-full + `docker compose build` |

---

## 7. Build order

The harness comes before most of the product, because it is what makes the product cheap to
build. Tracked as `HR-*` in [`BACKLOG.md`](BACKLOG.md).

| Step | Output | Why here |
|---|---|---|
| 0 | Prove the loop: audio → transcript → macros, one script, no infra | Answers whether the product works at all, for a day of effort. Nothing else is worth building if this is mush |
| 1 | `docs/specs/*` filled in from the settled model | Everything reads it. Fixing a spec is ~100× cheaper than fixing generated code |
| 2 | `check-invariants.ts` + hooks (INV-1,2,5,8 first) | Guardrails before the first line of app code |
| 3 | Scaffold, `/health`, `docker compose up` green, CI step 1 | The first gate |
| 4 | `evals/` + seed generator + 30 golden cases from real history | The one artifact only the owner can produce. Do it while the repo is empty |
| 5 | Skills, agents, `/slice` `/eval` `/promptdiff` | The loop closes |
| 6 | `/slice 1` … `/slice 9` | The actual product |

# Invariants

Eight rules. Each is architecture rather than style, each is mechanically checkable, and each
is checked by `scripts/check-invariants.ts` — on every file edit via a hook, and again in CI.

**Why a script and not a prompt.** Every "the agent must never…" line in a design doc is a bug
waiting to happen. Prompts are advisory; a failing exit code is not. This file is the reason it
is safe to let an agent write code quickly.

| # | Rule | Why it matters | How it is checked |
|---|---|---|---|
| INV-1 | Only `src/gateway/` may import a model SDK (`@anthropic-ai/sdk`, any STT client) | The whole cost-logging, routing and spend-cap story depends on one chokepoint | AST import scan |
| INV-2 | Every Mongo query filters by `user_id` | Multi-tenant health data. One missed scope leaks another person's medical records | AST call inspection on `.find/.findOne/.updateOne/.deleteOne/.aggregate`. Escape hatch: `// inv-2: ok <reason>` |
| INV-3 | Every `taskName` passed to the gateway exists in `TASK_ROUTES` | A typo is a runtime throw inside a 23:30 cron job with nobody watching | Collect string literals, diff against the route map |
| INV-4 | `/webhook/*` routes carry the signature-validation preHandler | Without it the endpoint is public and writable by anyone | AST route-registration scan |
| INV-5 | `services/api` and `services/dashboard` never import each other; the dashboard imports only `packages/contracts` | The one hard boundary in the system, and the thing that makes parallel work safe | Path + import scan |
| INV-6 | Prompt constants are module-level and never interpolated with user text | WhatsApp input is untrusted. This is the prompt-injection surface | AST: flag template literals and `.replace()` on `*_PROMPT` / `*_SYSTEM` identifiers |
| INV-7 | No `new Date()` day arithmetic outside `src/core/time.ts` | The server is UTC, the user is not. A 1 a.m. session belongs to yesterday's training day | AST call scan |
| INV-8 | No secret-shaped literal anywhere (`sk-ant-`, `AC[0-9a-f]{32}`, long base64) | Credential leak into a public MIT repo | Regex |

## Escape hatches

INV-2 is the only rule with one, because legitimate cross-user queries exist — the global
`exercise_library`, admin health checks, migrations. The comment must state a reason:

```ts
// inv-2: ok — exercise_library scope:"global" rows are shared by design
```

A bare `// inv-2: ok` fails. If a rule needs an escape hatch more than twice, the rule is wrong.

## Hook wiring

Not enabled yet — a `PostToolUse` hook calling a script that does not exist would fail on every
edit. Enable together with `HR-1`:

- `PreToolUse` on `Write|Edit` → deny paths matching `**/.env`, `infra/certs/**`, `**/*.pem`
- `PostToolUse` on `services/**/*.ts` → `biome check --write` then `check-invariants.ts <file>`
- `Stop` → `vitest run --silent` and a one-line summary

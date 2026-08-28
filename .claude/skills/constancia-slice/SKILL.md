---
name: constancia-slice
description: The order and definition-of-done for implementing a Constancia vertical slice. Use when implementing a spec, adding a feature, or when a change touches more than one layer.
---

# Implementing a slice

Read `docs/specs/NN-*.md` first. It is the instruction. If it is wrong or ambiguous, say so,
state your assumption, and continue — do not stop and wait unless proceeding would be unsafe.

## Order

Bottom-up, because each step is the contract for the next.

1. **Contract** — Zod schema in `packages/contracts`. Both services depend on this, so it moves first.
2. **Model + indexes** — collection shape from `docs/schema/collections.json`, index registered in `ensureIndexes()`.
3. **Gateway task** — if it calls a model, register `taskName` in `TASK_ROUTES` (see `constancia-gateway-task`).
4. **Logic** — the actual behaviour, in the narrowest module that can hold it.
5. **Route** — thin. Validation from the contract, no business logic in the handler.
6. **Tests** — happy path plus one failure path. Not coverage theatre; the failure path is the point.
7. **Eval case** — mandatory if the slice touched resolution, a prompt, or an agent.
8. **ADR** — only if you deviated from the spec on a decision, not a detail.

## Done means all of

- `pnpm typecheck` clean
- `pnpm test` green
- `node scripts/check-invariants.ts` passes
- `packages/contracts` regenerated if the HTTP surface changed
- an eval case exists if AI behaviour changed
- deviations named in your summary so `spec-keeper` can pick them up

## Traps in this codebase

- MongoDB cannot store a bare date. Use `date_key: string` as `"YYYY-MM-DD"`, user-local.
- Day boundaries are per-user timezone. Never `new Date()` arithmetic outside `src/core/time.ts`.
- The scheduler must not run in the web process — production runs multiple workers and every job would fire once per worker.
- Twilio retries anything that does not answer fast. Ack first, process after, dedupe on `provider_message_id`.
- Cost logging must never break a request that already succeeded. Wrap it.

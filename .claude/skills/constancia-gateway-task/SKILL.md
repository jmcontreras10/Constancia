---
name: constancia-gateway-task
description: How to add an AI or speech-to-text call in Constancia. Use whenever code needs to call a model — a new agent, a parser step, a transcription, a chat feature.
---

# Adding a model call

Adding an LLM call is a five-step ritual, not a one-liner. The ritual is what keeps cost,
routing and quality observable instead of scattered.

## Steps

1. **Register the task.** Add `taskName` to `TASK_ROUTES` in `src/gateway/routes.ts` with a
   complexity tier and, if relevant, `requiresVision`. INV-3 fails the build if a `taskName`
   reaches the gateway without being registered.
2. **Pick the tier honestly.** Frequency × stakes. A per-message classifier runs hundreds of
   times a day and should be cheap. `orchestrator_synthesis` runs once per user per week and is
   the output people actually read — spend there.
3. **Note expected volume** in a comment: calls per user per day. This is what makes the cost
   report interpretable later.
4. **Add an eval case.** A new AI call with no case is an untested deploy.
5. **Confirm it logs.** Every call lands in `model_calls` with `user_id`, `taskName`, cost and
   latency. STT calls use `provider: "stt"` and `audio_seconds` — one ledger, so the per-user
   spend cap is not half-blind.

## Rules

- **Only `src/gateway/` imports a model SDK** (INV-1). Everything else calls `router.complete()`.
- **Use structured outputs**, not a "respond in JSON" instruction. A parse failure inside a
  23:30 cron job with nobody watching is the worst place to discover a shimmed constraint.
- **Do not lowball `maxTokens`.** A truncated weekly review is silent garbage. Stream anything
  long.
- **Parse tool and JSON output**, never string-match it. Escaping varies by model.
- **Check the spend cap before the call, not after.** Over the line, degrade — cheaper tier, or
  queue for the nightly batch. Never refuse outright.

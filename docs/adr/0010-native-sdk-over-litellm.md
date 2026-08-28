# 0010 — Native SDK by default, LiteLLM as an optional adapter

2026-08-27 · Accepted

## Context

Routing every AI call through LiteLLM was the first plan, for model-agnosticism. It came with
`response_format: {type: "json_object"}` — an OpenAI-ism that LiteLLM shims by injecting a
"respond in JSON" instruction rather than actually constraining output.

## Decision

Keep the `ModelRouter` abstraction — one chokepoint, cost logging, task→tier mapping. Change its
default backend to the native Anthropic SDK. LiteLLM becomes an optional adapter for the
bring-your-own-model self-hosting story.

## Consequences

A unified interface is a lowest-common-denominator surface, and the features it costs are the
ones this product depends on:

| Feature | Why Constancia wants it |
|---|---|
| Structured outputs | Every agent returns JSON. A shimmed instruction fails as a parse error inside a 23:30 cron job |
| Prompt caching | The dashboard chat resends the same profile and history prefix every turn |
| Adaptive thinking / effort | The weekly synthesis is exactly the cross-domain reasoning this is for |
| Batch API (50% off) | Weekly reviews and over-cap photo estimation are latency-insensitive |

The "model-agnostic" claim survives — arguably more honestly, since the abstraction is now ours
rather than borrowed. INV-1 gets more valuable: both `@anthropic-ai/sdk` and any adapter stay
confined to `src/gateway/`.

Model tiers were also stale. Corrected: drop date suffixes, and `orchestrator_synthesis` is the
one call worth the strongest tier — it runs once per user per week and is the output people
actually read.

# 0001 — TypeScript, not Python

2026-08-27 · **Accepted**

## Context

FastAPI + Python was the initial instinct, on the usual "best LLM ecosystem" reasoning. The
owner is fluent in TypeScript and Kotlin, and unfamiliar with Python for web development.

That ecosystem argument was strong in 2023. By 2026 it is much weaker for *application* code:
Anthropic ships a first-class TypeScript SDK with full feature parity, and Constancia does no
training, no numpy, no dataframe work. It is an HTTP service that calls an LLM and stores JSON.

## Decision

TypeScript across the whole repo.

The deciding argument was the owner's, not the one above: **more developers are fluent in
TypeScript than in Kotlin.** For an MIT project that wants outside contributors, the size of the
pool that can read and extend the code outweighs any language-quality preference.

- `services/api` — Fastify + Zod + the official MongoDB driver + BullMQ
- `services/dashboard` — Next.js App Router + MUI + Recharts
- `packages/contracts` — Zod schemas shared by both

## Consequences

**Gained.** One language means the HTTP boundary is typed at both ends — a contract change
becomes a compile error rather than a runtime surprise. One toolchain: one linter, one test
runner, one type checker. And the owner writes at full speed instead of learning a stack.

**Given up.** Python's ML tooling, which this project does not use. Some LLM examples and blog
posts are Python-first and need translating.

**Cost of being wrong.** Low while no code exists. High after slice 3.

## Alternatives

**Kotlin** — better language, and the owner is fluent. Rejected because it puts two languages in
one repo, gives up shared types with the Next.js dashboard, and reaches Anthropic through the
Java SDK rather than an idiomatic client.

**Python as specified** — rejected on velocity. A marginal ecosystem edge does not beat writing
in a language you know, on a two-person project.

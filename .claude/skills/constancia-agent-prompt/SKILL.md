---
name: constancia-agent-prompt
description: House style for Constancia product-agent and resolver prompts. Use when creating or editing anything under src/agents/ or src/resolve/, or writing a judge rubric.
---

# Writing a product agent prompt

**A prompt edit is the highest-risk change in this repo and the only one with no signal in the
diff.** The code compiles, the tests pass, and the output quietly gets worse. Nothing here is
optional.

## Rules

- **Module-level constants.** `const EXERCISE_SYSTEM = \`...\`` — never built at call time.
- **Never interpolate user text into a prompt** (INV-6). WhatsApp input is untrusted and this is
  the injection surface. User data goes in a `user` message, delimited, marked as data.
- **Agents receive typed fields, never `raw_input`.** The parser's job is to make text into
  structure; the agent's job is to reason over structure.
- **Active hard constraints go in first**, above goals and above the routine. If a doctor
  forbade knee impact, no amount of goal-chasing may override it.
- **State the output contract as a schema**, enforced with structured outputs — not as a
  paragraph asking nicely for JSON.

## Every prompt ships with a rubric

`evals/agents/rubrics/<agent>.md`, scoring 1–5:

- **Groundedness** — does every numeric claim appear in the input? The one that matters most.
- **Actionability** — can the user do this tomorrow, or is it a platitude?
- **Constraint respect** — does it ever suggest something an active hard constraint forbids?
  A failure here is a hard stop regardless of other scores.
- **Cross-domain** (orchestrator only) — genuine correlation, or two reports concatenated?

## Before merging

Run `/promptdiff`. It runs the affected suite on `HEAD` and on the working tree and prints the
delta. **Do not merge a prompt change without it.** A drop in groundedness blocks regardless of
what else improved.

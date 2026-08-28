# Decision records

Why things are the way they are, and what was rejected. A spec says *what* to build; an ADR says
*why*, and what the alternative cost.

Write one when a choice would look arbitrary to someone reading the code in six months, or when
implementation deviates from a spec. Not for renamed variables.

| # | Decision | Status |
|---|---|---|
| [0001](0001-typescript-over-python.md) | TypeScript, not Python | Accepted |
| [0002](0002-mui-over-tailwind.md) | MUI, not Tailwind | Accepted |
| [0003](0003-plan-vs-history.md) | Plan and history are separate zones | Accepted |
| [0004](0004-routine-is-the-vocabulary.md) | The routine is the vocabulary | Accepted |
| [0005](0005-drafts-and-proposals.md) | Two conversation states, not zero | Accepted |
| [0006](0006-exercise-seed-source.md) | free-exercise-db as the seed | Accepted |
| [0007](0007-voice-first-input.md) | Voice is the primary meal input | Accepted |
| [0008](0008-meals-not-ingredients.md) | The unit is a meal, never an ingredient | Accepted |
| [0009](0009-constraint-vs-goal.md) | Conflicts are surfaced, never auto-resolved | Accepted |
| [0010](0010-native-sdk-over-litellm.md) | Native SDK by default | Accepted |
| [0011](0011-the-name.md) | Constancia, not FitCore | Accepted |

## Template

```
# NNNN — short title
date · Proposed | Accepted | Superseded by NNNN

## Context      what forced the decision
## Decision     what was chosen
## Consequences what it costs, and what it rules out
## Alternatives what was rejected, and why
```

---
name: constancia-eval
description: How to write, run and read Constancia evals. Use when adding a golden case, harvesting a production failure, interpreting an accuracy table, or changing the baseline.
---

# Evals

Full design: `docs/harness/evals.md`.

## Adding a case

`evals/parser/cases.jsonl`, one JSON object per line: the input exactly as a user sent it, and
the expected structured output.

- **Use real messages.** Invented ones test the parser against your imagination.
- **Include voice transcripts**, not just typed text — with the number errors STT actually makes.
- Preserve the invisible Unicode that WhatsApp copy-paste inserts. Stripping it while writing
  the case defeats the point.

## Harvesting a failure

Every production mis-parse becomes a case with the correct output. This is the flywheel and it
is why the resolver improves instead of drifting. Do this the day it happens, while you still
remember what the message meant.

## Running

```bash
npx tsx evals/run.ts parser     # field-level accuracy
npx tsx evals/run.ts agents     # rubric judges
npx tsx evals/run.ts all
```

## Reading a result

Compare against `evals/baseline.json`. Report **which cases newly fail and what they share** —
"set expansion dropped 3.6%" is not useful; "all four new failures are assisted pull-ups, where
the inverted scale value is being read as a weight" is.

Hard stops regardless of other columns: **intent classification** and **groundedness**.

## The baseline

`baseline.json` moves in its own commit, with a note explaining why. Moving it is a decision,
never a side effect of a passing run.

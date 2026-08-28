# Constancia docs

Everything here is written to be read by both people and agents. `CLAUDE.md` at the repo root
is the always-loaded contract; these are the details it points at.

| Path | What it is | Read it when |
|---|---|---|
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | The **build harness** — how agents, skills, hooks and pipelines fit together | Before touching the harness, or wondering why a rule exists |
| [`schema/collections.json`](schema/collections.json) | **Source of truth** for the data model. 15 collections, fully typed | Any time you touch data |
| [`BACKLOG.md`](BACKLOG.md) | Every tracked item with priority and status | Deciding what to do next |
| [`specs/`](specs/) | The build plan, one file per slice | Before implementing anything |
| [`adr/`](adr/) | Why decisions were made, and what was rejected | When something looks arbitrary |
| [`harness/invariants.md`](harness/invariants.md) | The eight hard rules, and how each is checked | Writing or fixing the invariant checker |
| [`harness/evals.md`](harness/evals.md) | How AI output quality is measured | Touching a prompt, parser or agent |

## The two source-of-truth rules

**The data model lives in `schema/collections.json`.** The Miro board is a rendering of it for
humans to argue over — never the other way round. When they disagree, the JSON wins and the
board gets updated. `HR-6` in the backlog is the generator that will make this mechanical.

**Prose lives in specs, decisions live in ADRs.** A spec says what to build. An ADR says why,
and what was rejected. If you find yourself explaining a rejected option inside a spec, it
wanted to be an ADR.

## Reading order, cold

1. `CLAUDE.md` — the contract
2. `ARCHITECTURE.md` §1–2 — what the harness is for
3. `schema/collections.json` — skim the `zones` and `summary` fields, not every column
4. `BACKLOG.md` — where things stand

The data model is 15 collections but only **eight hold durable data**. The `derived` zone is
recomputable and the `intake` zone TTLs itself out. Read those eight first.

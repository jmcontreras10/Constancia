# Probe

Step 0 of the build order. **Throwaway** — no database, no Twilio, no framework. Delete it once
it has done its job.

It answers the only four questions worth asking before writing real code:

1. Is Spanish speech-to-text good enough on food and gym vocabulary?
2. Are the macro estimates close enough to trust?
3. Does echoing the quantities back actually catch transcription errors?
4. **What does one meal cost?** — the number every later decision depends on.

## Run

```bash
cd scripts/probe && npm install
export ANTHROPIC_API_KEY=...
export STT_API_KEY=...
npx tsx probe.ts nota.ogg
```

Text-only, to test the estimation half without any audio:

```bash
npx tsx probe.ts --text "desayuno: 3 huevos, 2 tostadas integrales y un café con leche"
```

Sweep the tiers — this is the point, not an afterthought:

```bash
for m in claude-opus-5 claude-sonnet-5 claude-haiku-4-5; do
  npx tsx probe.ts nota.ogg --model "$m"
done
```

## STT provider

Defaults to a Groq-hosted Whisper endpoint because it is OpenAI-compatible, so any other
compatible provider is a base-URL change:

```bash
export STT_BASE_URL=https://api.openai.com/v1
export STT_MODEL=whisper-1
```

Language is **pinned** to `es` via `CONSTANCIA_LANG`, never auto-detected — mixed Spanish and
English breaks detection (ADR-0007).

## What to look at

- **The transcript, before anything else.** If numbers come back wrong, nothing downstream can
  recover. `ciento cincuenta` heard as `sesenta` is a 60% protein error that looks entirely
  plausible in the output.
- **`understood[]`** — did it echo the quantities you actually said, or invent them?
- **`basis`** — `weighed` only when you gave real amounts. If it says `weighed` for a vague
  description, the prompt is too eager.
- **`uncertain[]`** — an empty list on a genuinely vague meal means it is overconfident.
- **cost** — multiply by your realistic meals per day, then by 30.

## Caveats

- **Not typechecked.** Deps were not installed while writing it. Run `npm install` then
  `npx tsc --noEmit` before trusting the types.
- Pricing in `probe.ts` is approximate and hardcoded. Verify before drawing conclusions.
- STT cost is not counted — only the LLM call. Add it from your provider's dashboard.

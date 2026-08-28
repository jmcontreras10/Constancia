# 0007 — Voice is the primary meal input

2026-08-27 · Accepted

## Context

The owner already logs meals as WhatsApp voice notes in another tool and finds it accurate and
pleasant. Photos were the assumed rich input until then; audio had not been considered.

## Decision

Rank meal input: **template name → voice note → typed text → plate photo.**

A dedicated audio agent transcribes on arrival using a hosted STT API behind a swappable
adapter. It transcribes and stops — it never interprets.

## Consequences

The real argument for voice is not convenience, it is **what people say versus what they type**.
You type "desayuno". You say "desayuno: tres huevos, dos tostadas integrales y un plátano".
Speaking is cheap, so quantities get spoken that would never be typed — which does more for
estimation accuracy than any nudge.

It is also the cheapest rich input: transcription costs far less than a vision call on a plate,
and is more accurate, because no image distinguishes 120g of rice from 180g.

The audio never reaches the LLM. Transcription happens once, the transcript lands on
`messages.media[]`, and everything downstream is the existing text path — one step at the front,
zero changes behind it, one interpretation path to debug.

**The failure mode needs designing for.** Speech-to-text fails on numbers, and numbers are the
entire payload. Confirmations must echo the quantities heard — "escuché 150g pollo, 180g arroz
→ 520 kcal" — not just the computed result. Language is pinned from the user profile, never
auto-detected, because mixed Spanish and English breaks detection.

STT calls log to `model_calls` with `provider: "stt"` and `audio_seconds`, so one ledger covers
both LLM and transcription and the per-user spend cap is not half-blind.

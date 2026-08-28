# 0008 — The unit is a meal, never an ingredient

2026-08-27 · Accepted

## Context

Explicit owner constraint: no record per grape, per egg, per slice. Every existing tracker
fails at exactly this point, and the friction is why logging stops.

## Decision

`meals` stores a description and macro totals. There is no `items[]`, no ingredient breakdown,
not even optionally. `meal_templates` holds reusable dishes — meal prep, a specific restaurant
dish — with macros already worked out.

Templates emerge from repetition or are named outright ("de ahora en adelante, desayuno clásico
será…"). They are always user-scoped: exercises are universal, food is personal.

## Consequences

**Macros are copied onto the meal, not read through the template.** Correcting a template must
never silently rewrite months of logged history. Same rule as `routine_ref` against immutable
routines.

`portions: float` handles variation — "doble" is 2, "media porción" is 0.5 — without a second
estimation call.

`basis` records why a number is what it is: `template | weighed | described | photo_estimate |
user_stated`. More useful than a confidence float, because it says where the number came from
and lets a chart grey out the guesses.

**Precision is rewarded, never required.** A vague meal is logged immediately with a wide
estimate, then optionally refined. Logging never blocks on a question, and the nudge is capped
at one per day. The one place it is worth asking properly is template creation, because you pay
once and reuse forever.

The moment an ingredient field exists, something will start asking the user to fill it in. So it
does not exist.

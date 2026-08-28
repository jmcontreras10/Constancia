# 0002 — MUI, not Tailwind

2026-08-27 · Accepted

## Context

Tailwind with no component library — every control built from utility classes — was the first
instinct. Constancia's dashboard is data-dense — charts, tables, forms, settings — and is
built by two people plus agents.

## Decision

MUI for components. Recharts stays for charts.

## Consequences

**Gained.** Accessible, keyboard-navigable components for free, which matters most on the parts
nobody wants to hand-build: date pickers, tables, dialogs, form validation states. A theme
object is a better fit for agent-written code than a wall of utility classes, because the
palette lives in one typed place instead of being restated per element.

**Given up.** Bundle size, and less visual distinctiveness out of the box. The dark theme now
comes from a MUI theme rather than Tailwind config.

**Watch for.** Next.js App Router needs MUI's App Router integration package for SSR and
emotion cache. Getting this wrong produces a flash of unstyled content that is easy to miss
locally and obvious in production.

## Alternatives

**MUI X Charts instead of Recharts** — would unify the visual language. Not chosen: Recharts is
already specified, is lighter, and its composable API is easier for an agent to get right. Worth
revisiting if chart styling ends up fighting the MUI theme.

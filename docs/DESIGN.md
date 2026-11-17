# Design Direction & Experience Design

**Brand Identity:** **Timo** — **A little clarity for your digital day.** (**APPROVED — OWNER DECISION**)

## Core direction

Timo uses **Soft Playful Neo-Brutalism**. The interface combines warm neutral backgrounds, pastel accent surfaces, selective dark outlines, bold friendly typography, and small expressive illustrations. It feels friendly, gentle, calm, supportive, and playful without becoming childish.

## Composition rules

- Overview may use a bento/editorial composition: one dominant daily usage panel, a smaller active/idle split, top applications, category distribution, and a timeline preview.
- Timeline prioritizes time readability and chronology over decorative cards.
- Applications should feel like an organized library, not a marketplace.
- Application Details should prioritize metrics and usage history; mascot appearances should be contextual.
- Insights should use a small number of clear charts with explicit units and date ranges.
- Settings should be calm, conventional, and highly legible.

## Mascot placement

The mascot is a brand companion, not a permanent floating obstruction. Use it in onboarding, selected empty states, subtle welcome moments, and occasional contextual reactions. Do not place it over charts, tables, or frequently used controls. Provide a setting to reduce or hide mascot animation.

## Density and hierarchy

- Make the primary screen-time metric visually dominant.
- Separate active usage from idle and locked/sleep states.
- Keep labels concise and understandable.
- Avoid decorative notation, fake technical metadata, filler badges, and excessive pills.
- Use icons consistently from Lucide React; avoid mixing icon families.

## Motion philosophy

Motion should explain state changes and create feedback. It must not slow down navigation, distract during analysis, or run continuously in the tray. Follow `ANIMATION_SPECIFICATION.md`.

## Responsive window behavior

Although this is a desktop app, users resize windows. Design at a wide desktop target first, then support compact widths with collapsing sidebar/navigation and stacked content. Charts and tables must remain legible at common Windows scaling settings.

## Empty states

Use a helpful short explanation and one clear next step. Never show fabricated numbers. Example: “Timo is collecting your first activity sessions. Your usage summary will appear here.”

## Quality bar

The final UI should look intentional at 100%, 125%, and 150% Windows scaling. Check text clipping, focus visibility, chart labels, timeline overlap, and empty states. Visual audit screenshots are required before a milestone is considered complete.

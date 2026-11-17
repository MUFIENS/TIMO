# Style Guide — Timo UI Implementation

**Brand Voice & Tagline:** **Timo** — **A little clarity for your digital day.** (**APPROVED — OWNER DECISION**). Copy must stay warm, gentle, plain-language, non-judgmental, and avoid em dashes (`—`) inside rendered UI microcopy.

## Naming

- Use semantic token names (`bg-canvas`, `text-primary`, `border-strong`) rather than one-off color names.
- Component names use PascalCase; hooks start with `use`; Rust modules use snake_case.
- Do not encode state only in a color class. Add text/icon/accessible state.

## Tailwind and CSS

- Use Tailwind utility classes for layout and simple styling.
- Use CSS variables for theme tokens and semantic colors.
- Keep complex reusable visuals in component styles, not repeated long class strings.
- Avoid arbitrary one-off values unless the design has a documented reason.
- Do not use global CSS selectors that leak into third-party UI components.

## Component rules

- Prefer shadcn/ui and Radix primitives for accessible behavior.
- Customize components to the Timo system; do not leave default shadcn styling untouched if it conflicts with the brand.
- Keep component APIs small and typed.
- Separate visual components from data/query logic.
- Use controlled inputs when state is shared or persisted.
- Provide loading, empty, disabled, and error states.

## Icons and illustrations

- Use Lucide React consistently.
- Do not use emoji as primary UI icons.
- Use mascot/illustrations only where they add clarity or brand personality.
- Never replace the 3D mascot deliverable with a pile of SVG paths.

## Data display

- Format durations consistently (`2h 15m`; avoid ambiguous decimal hours).
- Use tabular numerals for changing values.
- Label date range and timezone context.
- Show zero, unavailable, and not-yet-collected states distinctly.
- Charts need accessible summaries or accompanying text.

## Interaction

- Buttons need clear action labels.
- Destructive actions require confirmation.
- Tooltips supplement labels; they do not replace essential instructions.
- Preserve keyboard focus and Escape behavior in dialogs.
- Respect reduced motion.

## Review checklist

- Is the visual hierarchy clear?
- Is there filler decoration or unnecessary badge/pill usage?
- Does the component work at 125% and 150% Windows scaling?
- Is it keyboard accessible?
- Are text contrast and focus indicators sufficient?
- Does motion have a purpose and a reduced-motion alternative?
- Are values derived from real data rather than mock placeholders?

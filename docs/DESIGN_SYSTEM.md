# Design System — Soft Playful Neo-Brutalism

## Identity

**Brand:** **Timo** — **A little clarity for your digital day.** (**APPROVED — OWNER DECISION**)  
**Design direction:** Soft Playful Neo-Brutalism.  
A friendly blend of soft pastel surfaces, expressive typography, structured outlines, editorial dashboard composition, and selective playful illustration.

## Visual principles

1. Playful without looking childish.
2. Bold without reducing readability.
3. Colorful without visual overload.
4. Structured without repetitive card grids.
5. Expressive without meaningless decoration.
6. Accessible before ornamental.

## Theme token palette (Provisional, WCAG 2.2 AA Math-Verified per `ADR-004`)

Use a three-layer token architecture (`primitive -> semantic -> component`) implemented via CSS custom properties (`:root` for Light, `.dark` for Dark). Visual verification at 100%, 125%, and 150% Windows scaling remains required during Phase 3/4.

### Light theme semantic tokens (`:root`)

| Semantic Token | Value | Usage & Contrast Pairing |
|---|---|---|
| `--bg-canvas` | `#FBF9F4` | Warm off-white app canvas |
| `--bg-surface` | `#FFFFFF` | Primary card and dialog surface |
| `--bg-surface-muted` | `#F1EFE9` | Secondary/sidebar surface |
| `--text-primary` | `#25232A` | Main text (`14.76:1` on canvas, `15.53:1` on surface, `13.50:1` on muted) |
| `--text-secondary` | `#5E5A64` | Supporting text (`6.39:1` on canvas, `6.73:1` on surface, `5.85:1` on muted) |
| `--text-inverse` | `#FFFFFF` | Text ONLY on solid dark/deep accents (`--accent-primary-solid`), NEVER on light pastels |
| `--border-strong` | `#29272B` | Selected Neo-Brutalist outlines and pastel chart/timeline bar strokes (`14.06:1` on canvas) |
| `--border-control` | `#88828E` | Interactive input/select boundaries (`3.73:1` on surface, `3.54:1` on canvas, >= 3:1 WCAG 1.4.11) |
| `--border-subtle` | `#E5E1D9` | Decorative hairlines and non-interactive dividers only (`1.24–1.30:1`) |

### Light theme accents & accessible ink pairings

Do not place `#FFFFFF` text on light pastel fills (`#FFFFFF` on `#A99AEF` is `2.45:1` FAIL; `#FFFFFF` on `#F38B78` is `2.39:1` FAIL). Pair pastel fills with `--text-primary` (`#25232A`), and use darkened `-ink` / `-solid` tokens for links, icons on white, solid CTAs, and focus rings:

| Token | Pastel Fill (`#25232A` text ratio) | Accessible Ink / Solid (`#FFFFFF` bg or fg ratio) | Suggested Use |
|---|---|---|---|
| `lavender` | `#A99AEF` (`6.33:1`) | `--accent-primary-solid` `#5A46B5` (`7.05:1` on/with white) | Primary playful fill (`#25232A` ink) & solid CTA/focus ring (`#5A46B5`) |
| `mint` | `#A9D9B8` (`9.84:1`) | `#1E6B43` (`6.48:1`) | Positive/active indicator & Productivity category |
| `blue` | `#83BCEB` (`7.66:1`) | `#1F5E8E` (`6.89:1`) | Supporting accent & Entertainment category |
| `butter` | `#FFDA77` (`11.49:1`) | `#7A5900` (`6.45:1`) | Highlight/information & Communication category |
| `pink` | `#F2A5BE` (`8.06:1`) | `#99325A` (`7.10:1`) | Secondary playful accent |
| `coral` | `#F38B78` (`6.49:1`) | `#B53D2C` (`5.72:1`) | Attention/warning/destructive ink |

### Dark theme semantic tokens (`.dark`, Warm Plum-Charcoal)

Designed intentionally without mechanical inversion (`ADR-004`):

| Semantic Token | Value | Usage & Contrast Pairing |
|---|---|---|
| `--bg-canvas` | `#18171C` | Warm plum-charcoal app canvas |
| `--bg-surface` | `#222028` | Elevated card/dialog surface |
| `--bg-surface-muted` | `#2B2832` | Sidebar and secondary surface |
| `--text-primary` | `#F5F2EC` | Main text (`15.95:1` canvas, `14.40:1` surface, `12.94:1` muted) |
| `--text-secondary` | `#B8B2C2` | Supporting text (`8.65:1` canvas, `7.80:1` surface, `7.01:1` muted) |
| `--border-strong` | `#D7D1E0` | Selected Neo-Brutalist outlines (`10.79:1` on surface) |
| `--border-control` | `#888196` | Interactive control borders (`4.31:1` surface, `4.78:1` canvas) |
| `--border-subtle` | `#3D3847` | Decorative hairlines only (`1.42:1` on surface) |
| `--focus-ring` | `#B6A8F5` | Focus outline (`8.41:1` canvas, `7.59:1` surface, `6.82:1` muted) |

Dark-mode category/brand pastel fills (`--ink-on-pastel: #18171C`, also >= 6.77:1 against `#222028` for icons/chart lines): `lavender` `#B6A8F5` (`8.41:1`), `mint` `#9BD1AC` (`10.28:1`), `blue` `#84B9E6` (`8.54:1`), `butter` `#F2CE6D` (`11.73:1`), `pink` `#EB9FB8` (`8.64:1`), `coral` `#EF8E7C` (`7.50:1`).

## Category colors

Use stable semantic tokens rather than hard-coded colors in individual components. A category must also have an icon/label/pattern so it is not distinguished by color alone.

| Category | Initial accent |
|---|---|
| Productivity | Mint |
| Entertainment | Blue |
| Gaming | Lavender |
| Communication | Butter/amber |
| System & Utilities | Neutral teal/gray |
| Others/Uncategorized | Neutral gray |

## Typography

- Primary UI: **Plus Jakarta Sans** (preferred) or Nunito Sans after legibility testing.
- Display/large statistic option: **Space Grotesk**.
- Technical metadata only: **JetBrains Mono**.
- Use tabular numerals for durations and statistics.
- Keep body text comfortable; default target 14–16 px at 100% scale, with user/system scaling respected.

## Shape and borders

- Cards: 14–20 px radius depending on hierarchy.
- Inputs/buttons: consistent 10–14 px radius.
- Select cards may use 1–1.5 px charcoal outlines.
- Use strong outlines selectively, not around every element.
- Shadows should be soft, restrained, and never substitute for clear boundaries.

## Layout

- Desktop-first responsive grid, persistent sidebar, generous whitespace.
- Use bento/editorial compositions for Overview, but maintain predictable alignment.
- Avoid identical three-column card rows on every page.
- Use charts only when they answer a real user question.
- Ensure useful content is visible without forcing excessive scrolling.

## Components

Build from shadcn/ui and Radix primitives, then theme to Timo. Core components: Button, IconButton, Card, StatCard, AppIcon, CategoryTag, NavigationItem, Select, Dialog, Tooltip, Tabs, DateRangePicker, EmptyState, ErrorState, Skeleton, UsageTimeline, CategoryLegend, ChartTooltip, AppReviewDialog.

## Themes

Light theme (`:root`) is the primary design reference. Dark theme (`.dark`) uses the semantic warm plum-charcoal tokens defined above (`ADR-004`) rather than mechanical inversion, preserving category meaning and WCAG 2.2 AA contrast in both themes.

## Accessibility

- WCAG 2.2 AA contrast target (`>= 4.5:1` normal text, `>= 3.0:1` large text and non-text UI boundaries/focus indicators; see `ADR-004`).
- Visible keyboard focus (`2px` solid `--focus-ring` with `2px` offset on `:focus-visible`; never `outline: none` without replacement).
- Distinct hover, active, disabled (`disabled` / `aria-disabled="true"`, `cursor: not-allowed`, `opacity: 0.5`), loading, and error states.
- Accessible names for icon-only buttons; `cursor: pointer` on enabled interactive controls.
- Keyboard-operable dialogs, menus, charts (plus data-table/summary alternatives).
- Reduced motion (`<MotionConfig reducedMotion="user">` synchronized with in-app setting; `[data-theme-switching] * { transition: none !important }` during theme flip).
- Never communicate status or category by color alone (always pair with Lucide icon, text label, or pattern/stroke).
- Support Windows display scaling (`100%`, `125%`, `150%`) and text scaling.

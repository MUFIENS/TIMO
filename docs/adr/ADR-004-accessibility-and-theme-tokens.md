# ADR-004: Accessibility, Contrast Compliance & Light/Dark Theme Tokens

**Date:** 2026-11-17  
**Status:** Provisional (Light & Dark palette math verified via `antislop-human/contrast-check.py`; visual DPI audit at 100%/125%/150% pending Phase 3/4 UI build)  
**Owners:** UI/UX Design System Engineering, Accessibility Engineering  

---

## 1. Context

Timo's approved visual direction is **Soft Playful Neo-Brutalism** (`DESIGN.md`, `DESIGN_SYSTEM.md`, `STYLEGUIDE.md`), featuring warm off-white surfaces, pastel category accents (`lavender`, `pink`, `mint`, `butter`, `coral`, `blue`), and selective dark charcoal outlines. `DESIGN_SYSTEM.md` explicitly marked its initial 14 light-mode tokens as provisional and required WCAG 2.2 AA validation, while providing zero dark-mode hex values.

## 2. Problem

Running `antislop-human/contrast-check.py` against the provisional tokens in `DESIGN_SYSTEM.md` exposed concrete WCAG 2.2 AA failures:
1. `#FFFFFF` (`text-inverse`) on `#A99AEF` (`lavender`) = **2.45:1** (**FAIL** 4.5:1 normal & 3.0:1 large).
2. `#FFFFFF` (`text-inverse`) on `#F38B78` (`coral`) = **2.39:1** (**FAIL** 4.5:1 normal & 3.0:1 large).
3. `#E5E1D9` (`border-subtle`) vs `#FBF9F4` (`1.24:1`) and `#FFFFFF` (`1.30:1`) **FAIL** WCAG 1.4.11 (`3.0:1` UI component boundary contrast) if used as input/control borders.
4. Borderless pastel bars/segments (`mint` `1.58:1`, `butter` `1.35:1`, `blue` `2.03:1`, `lavender` `2.45:1`) vs `#FFFFFF` **FAIL** WCAG 1.4.11 `3.0:1` non-text contrast.
5. `text-secondary` (`#6C6872`) on `#F1EFE9` (`surface-muted`) was `4.73:1` (slim margin near Windows ClearType/scaling).
6. No dark-theme semantic tokens existed (`DESIGN_SYSTEM.md` §"Themes").

## 3. Options Considered

1. **Option A: Replace Timo's pastel palette with high-saturation dark colors** (destroys Soft Playful Neo-Brutalism).
2. **Option B: Preserve Soft Playful Neo-Brutalism using a 3-layer token architecture (`Primitive -> Semantic -> Component`), pairing pastel surfaces strictly with dark charcoal ink (`#25232A`), adding accessible solid/text ink tokens (`#5A46B5`, etc.), enforcing Neo-Brutalist `border-strong` (`#29272B`) / `--border-control` (`#88828E`) strokes for 3:1 non-text boundaries, and authoring a warm plum-charcoal Dark Theme.**

## 4. Technical Trade-offs

- **Option B** preserves 100% of Timo's approved pastel identity (`#FBF9F4`, `#A99AEF`, `#A9D9B8`, `#FFDA77`, `#83BCEB`, `#F2A5BE`, `#F38B78`) while achieving measured WCAG 2.2 AA compliance across text, borders, charts, timeline blocks, and focus rings in both themes.
- Requires discipline: developers must **never** place `#FFFFFF` text onto pastel fill tokens (`--accent-lavender-fill`, `--accent-coral-fill`), and chart/timeline bars on light cards must carry a `1-1.5px` dark outline (`--border-strong`) or solid ink series accent.

## 5. Recommended Decision

Adopt **Option B** using three-layer tokens (`primitives` -> `semantic (:root / .dark)` -> `component`):

### Verified Light Theme Tokens (Measured via `antislop-human/contrast-check.py`)

| Semantic Token | Hex | Paired Against | Measured Ratio | WCAG 2.2 AA Verdict |
|---|---|---|---:|---|
| `--bg-canvas` | `#FBF9F4` | `--text-primary` (`#25232A`) | 14.76:1 | PASS (Normal & Large) |
| `--bg-surface` | `#FFFFFF` | `--text-primary` (`#25232A`) | 15.53:1 | PASS (Normal & Large) |
| `--bg-surface-muted` | `#F1EFE9` | `--text-primary` (`#25232A`) | 13.50:1 | PASS (Normal & Large) |
| `--text-secondary` (refined) | `#5E5A64` | `surface` / `canvas` / `surface-muted` | 6.73 / 6.39 / 5.85:1 | PASS (Normal & Large) |
| `--border-strong` | `#29272B` | `canvas` (`#FBF9F4`) | 14.06:1 | PASS (Non-text >= 3:1) |
| `--border-control` (new) | `#88828E` | `surface` / `canvas` | 3.73 / 3.54:1 | PASS (Non-text >= 3:1) |
| `--border-subtle` (decorative only) | `#E5E1D9` | `canvas` / `surface` | 1.24 / 1.30:1 | Hairlines only; never sole input/control boundary |
| `--accent-lavender-fill` + `--ink-on-pastel` | `#A99AEF` + `#25232A` | Pill/button/tag fill | 6.33:1 | PASS (Normal & Large) |
| `--accent-primary-solid` + `--text-inverse` | `#5A46B5` + `#FFFFFF` | Solid primary CTA / links / focus ring | 7.05:1 (on white/inverse) / 6.70:1 (`canvas`) / 6.13:1 (`muted`) | PASS (Normal & Large + 3:1 Ring) |
| `--accent-mint-fill` / `--category-productivity-ink` | `#A9D9B8` / `#1E6B43` | `#25232A` on fill / `#1E6B43` on `#FFFFFF` | 9.84:1 / 6.48:1 | PASS |
| `--accent-blue-fill` / `--category-entertainment-ink` | `#83BCEB` / `#1F5E8E` | `#25232A` on fill / `#1F5E8E` on `#FFFFFF` | 7.66:1 / 6.89:1 | PASS |
| `--accent-butter-fill` / `--category-communication-ink` | `#FFDA77` / `#7A5900` | `#25232A` on fill / `#7A5900` on `#FFFFFF` | 11.49:1 / 6.45:1 | PASS |
| `--accent-pink-fill` / `--accent-pink-ink` | `#F2A5BE` / `#99325A` | `#25232A` on fill / `#99325A` on `#FFFFFF` | 8.06:1 / 7.10:1 | PASS |
| `--accent-coral-fill` / `--status-danger-ink` | `#F38B78` / `#B53D2C` | `#25232A` on fill / `#B53D2C` on `#FFFFFF` | 6.49:1 / 5.72:1 | PASS |

### Verified Dark Theme Tokens (Warm Plum-Charcoal, Measured via `contrast-check.py`)

| Semantic Token | Hex | Paired Against | Measured Ratio | WCAG 2.2 AA Verdict |
|---|---|---|---:|---|
| `--bg-canvas` (`.dark`) | `#18171C` | `--text-primary` (`#F5F2EC`) | 15.95:1 | PASS (Normal & Large) |
| `--bg-surface` (`.dark`) | `#222028` | `--text-primary` (`#F5F2EC`) | 14.40:1 | PASS (Normal & Large) |
| `--bg-surface-muted` (`.dark`) | `#2B2832` | `--text-primary` (`#F5F2EC`) | 12.94:1 | PASS (Normal & Large) |
| `--text-secondary` (`.dark`) | `#B8B2C2` | `canvas` / `surface` / `surface-muted` | 8.65 / 7.80 / 7.01:1 | PASS (Normal & Large) |
| `--border-strong` (`.dark`) | `#D7D1E0` | `surface` (`#222028`) | 10.79:1 | PASS (Non-text >= 3:1) |
| `--border-control` (`.dark`) | `#888196` | `surface` / `canvas` | 4.31 / 4.78:1 | PASS (Non-text >= 3:1) |
| `--border-subtle` (`.dark`, decorative only) | `#3D3847` | `surface` (`#222028`) | 1.42:1 | Hairlines only |
| `--focus-ring` / `--accent-lavender` (`.dark`) | `#B6A8F5` | `canvas` / `surface` / `muted` (or `#18171C` ink on fill) | 8.41 / 7.59 / 6.82:1 (fill+ink: 8.41:1) | PASS (Text & 3:1 Ring) |
| `--accent-mint` (`.dark`) | `#9BD1AC` | `surface` (`#222028`) / `#18171C` ink on fill | 9.28:1 / 10.28:1 | PASS |
| `--accent-blue` (`.dark`) | `#84B9E6` | `surface` (`#222028`) / `#18171C` ink on fill | 7.71:1 / 8.54:1 | PASS |
| `--accent-butter` (`.dark`) | `#F2CE6D` | `surface` (`#222028`) / `#18171C` ink on fill | 10.59:1 / 11.73:1 | PASS |
| `--accent-pink` (`.dark`) | `#EB9FB8` | `surface` (`#222028`) / `#18171C` ink on fill | 7.80:1 / 8.64:1 | PASS |
| `--accent-coral` (`.dark`) | `#EF8E7C` | `surface` (`#222028`) / `#18171C` ink on fill | 6.77:1 / 7.50:1 | PASS |

### Focus, Hover, Active, Disabled & Reduced-Motion Rules
- **Focus Ring (WCAG 2.4.7 & 2.4.11):** `:focus-visible` uses `outline: 2px solid var(--focus-ring)` with `outline-offset: 2px` (`#5A46B5` light >= 6.13:1; `#B6A8F5` dark >= 6.82:1), never `outline: none` without replacement.
- **Hover / Active / Disabled:** Hover shifts surface lightness/border emphasis while preserving >= 4.5:1 text contrast; active presses cleanly without shifting layout bounds; disabled uses explicit `disabled` / `aria-disabled="true"` + `cursor: not-allowed` + `opacity: 0.5` (`states-and-variants.md`) and avoids disabled clicks.
- **Theme Switching:** Apply `[data-theme-switching] * { transition: none !important }` during light/dark toggle (`ui-animation`) to eliminate transition flash.
- **Reduced Motion:** Wrap UI in `<MotionConfig reducedMotion="user">` synchronized with Timo's in-app `reduced_motion` preference (`ANIMATION_SPECIFICATION.md`), replacing spatial slides/bounces with brief opacity transitions (<= 150ms) and pausing 3D mascot loops.
- **Non-Color Redundancy (WCAG 1.4.1):** Every category pill, chart series, timeline block, and tracking status pairs color with a Lucide icon, text label, and/or stroke/pattern differentiation.

## 6. Decision Status

**Provisional** (contrast math verified via `contrast-check.py`; subject to visual audit across Windows 11 100%/125%/150% DPI scaling in Phase 3/4).

## 7. Consequences

- **Positive:** Eliminates all WCAG 2.2 AA contrast failures found during audit while keeping Soft Playful Neo-Brutalism intact across both light and dark modes.
- **Negative / Risk:** Dark-mode pastel strokes require restraint so outlines (`#D7D1E0`) do not overwhelm dense tables; use `--border-control` (`#888196`, 4.31:1) on inputs/tables and reserve `--border-strong` (`#D7D1E0`) selectively for Neo-Brutalist emphasis cards, primary controls, and chart bars.

## 8. Implementation Implications

- Implement tokens as CSS custom properties (`src/styles/tokens.css`) mapped cleanly to Tailwind CSS v4 `@theme` and shadcn/ui semantic variables (`--background`, `--foreground`, `--primary`, `--ring`, `--border`).
- Add base-layer `cursor: pointer` for enabled interactive controls (`antislop-human`).
- Update `DESIGN_SYSTEM.md` with the verified light refinements and complete dark theme token table.

## 9. Validation Requirements

1. Automated contrast verification in CI/pre-release checks for both `.light` and `.dark` themes.
2. Keyboard-only Tab/Shift+Tab/Enter/Space/Escape audit + Windows 100%, 125%, 150% DPI visual screenshots (`QA_TESTING_PLAN.md`).
3. Reduced-motion toggle audit (`QA-015`).

## 10. Related Documents

- [`DESIGN_SYSTEM.md`](../DESIGN_SYSTEM.md)
- [`DESIGN.md`](../DESIGN.md)
- [`STYLEGUIDE.md`](../STYLEGUIDE.md)
- [`ANIMATION_SPECIFICATION.md`](../ANIMATION_SPECIFICATION.md)
- [`QA_TESTING_PLAN.md`](../QA_TESTING_PLAN.md) (`QA-015`)

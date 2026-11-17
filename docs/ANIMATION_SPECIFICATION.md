# Animation Specification — Timo

## Motion principles

Motion should clarify cause and effect, provide feedback, and add warmth. It must not create visual noise or measurable tracking overhead. Respect Windows reduced-motion preferences and the in-app setting.

## UI motion categories

| Category | Examples | Initial timing guidance |
|---|---|---|
| Micro-interaction | Button press, toggle, hover, tooltip | 120–180 ms |
| Component transition | Dialog, dropdown, expanding panel | 160–240 ms |
| Page transition | Route/content transition | 180–280 ms |
| Data reveal | Chart entrance, stat count transition | 250–500 ms; avoid replay on every render |
| Mascot gesture | Greet, curious, celebrate | Clip-specific; generally brief and restrained |

Durations are guidelines, not hard rules. Motion must feel responsive and be validated at 60 Hz and under reduced performance.

## Implementation library

Use Motion for React for UI transitions and interactions. Keep CSS transitions for simple hover/focus effects. Avoid introducing a second general-purpose UI animation library without an ADR.

## Mascot state machine

Suggested states:
- `hidden`
- `idle`
- `greeting`
- `curious`
- `happy`
- `relaxed`
- `celebrating`

Events can trigger transitions, but do not replay long reactions repeatedly for noisy tracking events. Debounce/coalesce repeated recognition events.

## State mapping

| Application event | Mascot response |
|---|---|
| First successful onboarding | `greeting` once |
| Default visible state | `idle` |
| Unknown app enters review queue | Optional brief `curious` reaction, rate-limited |
| User completes a self-defined goal (future) | `happy` / `celebrating` |
| User opens a break reminder (future) | `relaxed` |
| Mascot hovered/clicked | Small look/reaction, never blocks controls |

No negative or shaming reaction should be tied to gaming or entertainment time.

## Performance rules

- Lazy-load model and textures.
- Pause the render loop when hidden, minimized, off-screen, or in the tray.
- Do not render a full 3D scene in every page if a single shared mascot component can be used.
- Avoid continuous post-processing and expensive shadows.
- Reuse materials and dispose GPU resources on unmount where appropriate.
- Use a static loading poster only during load/error fallback.
- Reduced-motion mode must disable loops and use minimal or no transitions.

## Accessibility

Motion must never be the only way to communicate state. Provide text/icon alternatives. Avoid flashing, large screen movement, and long forced animations. Respect `prefers-reduced-motion` and application setting.

## QA

Test animation clips, interruption, rapid state events, hidden window, minimize-to-tray, app shutdown, low-end integrated graphics, and reduced-motion mode.

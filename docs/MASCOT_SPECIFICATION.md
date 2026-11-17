# Mascot Specification — Timo Companion

## Purpose

The mascot gives Timo a recognizable, warm identity and creates gentle emotional feedback. It must never become an obstacle to reading analytics or operating the application.

## Character direction

- **Type:** original stylized 3D fantasy creature; not a human, generic robot, or copied existing character.
- **Silhouette:** compact, rounded, soft, slightly chubby, instantly recognizable at small sizes.
- **Face:** large expressive eyes, simple mouth, readable brows/eye shapes, friendly proportions.
- **Limbs:** short and simple; enough articulation for gentle gestures.
- **Surface:** soft matte or restrained satin/PBR material; avoid excessive plastic shine.
- **Palette:** primary blue/lavender family with restrained mint/cream accents. Final colors must harmonize with the Timo UI palette.
- **Personality:** friendly, curious, calm, supportive, quietly playful.

## Originality

Do not replicate a known mascot, commercial character, game character, or reference image one-to-one. Use reference images only to communicate broad style and quality. The final design must have an original silhouette, face, proportions, and color arrangement.

## Required views and deliverables

1. Front, 3/4, side, and back concept views.
2. Expression sheet: neutral, happy, curious, relaxed, attentive, gentle reminder.
3. Production 3D model with clean topology appropriate to its deformation needs.
4. UVs and optimized PBR textures/materials.
5. Rig/skeleton suitable for the character's actual anatomy.
6. Named animation clips.
7. Optimized GLB export and a lightweight preview render.
8. Asset metadata: source tool, generation date, license/usage terms, dimensions, triangle count, texture sizes, animation list, and known limitations.

## Required animation clips

- `idle_breathe` — subtle loop.
- `blink` — short blink; may be procedural if rig supports it.
- `look_left`, `look_right`, `look_cursor` — eye/head orientation where supported.
- `greet` — short welcome gesture.
- `happy_react` — small positive reaction.
- `curious_react` — gentle head tilt/eye movement.
- `relaxed` — calm posture/breathing.
- `celebrate` — short, restrained celebration.

The exact names can be mapped during integration, but the exported manifest must list every clip and its duration.

## Interaction behavior

- Cursor tracking is subtle and clamped; no extreme eye rotation.
- On touch-capable systems, use click/tap rather than cursor assumptions.
- Hover does not trigger a long, noisy animation.
- Mascot can be hidden or reduced through accessibility settings.
- Respect reduced motion and pause rendering when not visible.

## Technical constraints

- Export GLB/glTF 2.0.
- Keep model and texture budgets appropriate for a small desktop utility; record actual metrics and optimize after profiling.
- Avoid unnecessary high-resolution textures, transparent layered materials, excessive bones, and expensive post-processing.
- Use named clips and predictable coordinate/orientation conventions.
- Include fallback behavior for missing or incompatible animation clips.

## Acceptance criteria

- Character is original and matches approved concept references.
- Silhouette and face remain readable at small UI sizes.
- Rig deforms without visible collapsing in the required motions.
- All required clips play correctly in the target Three.js/R3F setup.
- No severe mesh artifacts, material errors, or missing textures.
- Rendering pauses when hidden or reduced-motion mode is enabled.
- Agent provides the actual GLB and preview; a static image alone is not acceptance.

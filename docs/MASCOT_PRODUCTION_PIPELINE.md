# Mascot Production Pipeline — AI-Agent Owned

## Ownership rule

The AI development agent owns the complete mascot production pipeline. The product owner is not expected to open Blender, model, rig, or animate manually. The agent must use available image-generation, 3D-generation, command-line, Blender automation, and validation tools as appropriate.

**Honesty requirement:** never state that modeling, rigging, or animation succeeded unless the resulting asset exists and was inspected. If a service requires an unavailable API key, paid credit, subscription, license, or human-only step, report the exact blocker and continue with any viable alternative. Do not silently substitute SVG or a static illustration for the required 3D model.

## Pipeline stages

### Stage 1 — Concept exploration
- Read `MASCOT_SPECIFICATION.md`, `DESIGN_SYSTEM.md`, and `ANIMATION_SPECIFICATION.md`.
- Create several original concepts with front/three-quarter views and consistent proportions.
- Compare silhouette, expression readability, and suitability for rigging.
- Select one concept for refinement; record why.

### Stage 2 — Multi-view reference set
- Generate consistent front, side, back, and three-quarter views.
- Keep eye placement, body proportions, accessories, colors, and silhouette consistent.
- Do not use four unrelated generations as a fake turnaround.

### Stage 3 — 3D generation
- Evaluate current tools such as Tripo or Meshy using their current official documentation and terms.
- Prefer image-to-3D or multi-view-to-3D when a controlled concept has been approved.
- Generate a small number of candidates and inspect geometry/material quality.
- Preserve source prompts, task IDs, license terms, and generation provenance.

### Stage 4 — Automated refinement
- Use Blender in headless/background mode or another available 3D pipeline when possible; the user does not operate the GUI.
- Check normals, manifold issues, topology, UVs, material assignments, scale, origin, and orientation.
- Retopologize/decimate only as needed; do not destroy deformation quality.
- Apply practical texture limits and remove unused geometry/materials.

### Stage 5 — Rigging
- Determine the correct skeleton for the character anatomy; do not assume humanoid rigging works for a fantasy creature.
- Run the provider's rig compatibility check when available.
- Validate joint placement, skin weights, eye controls, mouth controls, and limb deformation.
- If auto-rigging fails, use scripted/custom rigging or another provider. Log what was actually done.

### Stage 6 — Animation
- Create required clips from `MASCOT_SPECIFICATION.md`.
- Use real skeletal/morph animation where supported. Procedural eye motion may be used only when the model structure supports it and the effect is stable.
- Avoid repeated full-body bounces or distracting idle loops.
- Keep clips short and use easing appropriate to character weight.

### Stage 7 — Export
- Export GLB with embedded or correctly referenced textures and named clips.
- Validate file opens in a clean GLTF viewer and in the project runtime.
- Produce a `mascot_manifest.json` with asset path, clip names/durations, triangle count, texture dimensions, file size, rig details, and tool/license provenance.

### Stage 8 — Integration and optimization
- Load on demand using React Three Fiber/Three.js.
- Pause render loop when window is hidden, scene is off-screen, or reduced-motion is enabled.
- Use a poster/thumbnail only as loading fallback, never as the claimed final mascot.
- Profile CPU/GPU/memory impact on the reference machine.

## Tool preference

- Image concept: approved image-generation capability.
- 3D generation: evaluate Tripo/Meshy or equivalent current provider.
- Refinement/automation: Blender command line + Python scripts where available.
- Runtime: Three.js + React Three Fiber + Drei, GLB/GLTF.

## Provider capability & environment note (`ADR-007`)

As verified in Phase 0 (`ADR-007`, 2026-11-17):
- **Tripo vs Meshy:** Tripo OpenAPI (`v3.0`, `v3/animations/rig-check`) explicitly supports non-humanoid creature rigs (`quadruped`, `others`, etc.) and `.glb`/`.fbx` output, making it preferred over Meshy's humanoid-focused rigging API (`UE5`/`Unity`/`Mixamo`). However, Tripo's Free plan retains IP with Tripo and requires attribution; commercial desktop distribution requires a **Paid API plan** with prepaid credits (~25 credits/rig, ~10-20 credits/clip). Official reference: https://developers.tripo3d.ai/en/docs/animations-rig
- **Facial expressions & Blender dependency:** External auto-riggers focus on body bones and do not guarantee eye-blink, gaze, or mouth morph targets (`MASCOT_SPECIFICATION.md`). Headless Blender CLI + Python automation is therefore required for Stage 4 refinement, facial rigging, and GLB validation.
- **Current host status & staged plan (`ADR-007`, Approved Staged Plan):** As of 2026-11-17, `blender.exe`, `gemini-api` CLI, `TRIPO_API_KEY`, and `MESHY_API_KEY` are absent from the local environment. Per the owner-approved staged schedule, headless Blender and external 3D generation tooling are deferred until entering Phase 5 (Mascot Production); Phase 5 3D execution remains **Blocked** until those prerequisites are provisioned.

## Licensing and secrets

- Check tool and generated asset terms before distribution.
- Never commit API keys or tokens. Use environment variables or local secret storage.
- Keep provider credentials out of generated logs and manifests.
- Record attribution or license requirements in `THIRD_PARTY_NOTICES.md` if needed.

## Acceptance gate

Do not mark mascot production complete until the actual optimized GLB, manifest, required animation clips, visual preview, integration test, and performance check exist. If any required part is blocked, mark it BLOCKED with evidence and the next actionable requirement.

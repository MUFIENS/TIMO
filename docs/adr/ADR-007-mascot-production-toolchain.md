# ADR-007: Autonomous 3D Mascot Production Toolchain & Staged Preparation

**Date:** 2026-11-17  
**Status:** Approved (Owner approved staged preparation — defer headless Blender and 3D generation service setup until entering Phase 5) / Blocked (Phase 5 3D execution remains blocked until Phase 5 prerequisites are installed and provisioned)  
**Owners:** AI Development Agent (Pipeline Owner), Product Owner (Credentials/Approval)  

---

## 1. Context

Timo requires an original, friendly, compact stylized 3D fantasy creature mascot in Soft Playful Neo-Brutalist blue/lavender/mint/cream colors, rigged and animated across 10 named clips (`idle_breathe`, `blink`, `look_left`, `look_right`, `look_cursor`, `greet`, `happy_react`, `curious_react`, `relaxed`, `celebrate`), exported as an optimized `.glb` plus `mascot_manifest.json`, and lazy-loaded via Three.js + React Three Fiber + Drei (`MASCOT_SPECIFICATION.md`, `MASCOT_PRODUCTION_PIPELINE.md`, `ANIMATION_SPECIFICATION.md`). The product owner does **not** manually operate Blender, and static SVG cannot substitute for the final 3D mascot.

## 2. Problem

Phase 0 environment inspection on 2026-11-17 must verify which concept-art, 3D-generation, Blender automation, rigging, and licensing dependencies actually exist locally vs which require setup, credentials, or paid credits before Phase 5 begins.

## 3. Options Considered

1. **Concept Generation:** Antigravity `image-generator` subagent / Gemini Image API (`gemini-nano-banana-2.1` or `gemini-3-pro-image` via `gemini-api` CLI/SDK).
2. **3D Mesh & Rigging Provider:**
   - **Tripo 3D API (`v3.0`):** Supports non-humanoid creature rigs (`quadruped`, `others`, etc.), pre-rig compatibility endpoint (`v3/animations/rig-check`), animation retargeting, and `.glb` output. Free tier retains IP with Tripo and requires attribution; **Paid API plan** grants commercial ownership of private outputs (at ~25 credits/rig + ~10-20 credits/clip).
   - **Meshy API:** Supports `.glb` auto-rigging and animation presets, but primarily targets **humanoid / bipedal** rigs (`UE5 Mannequin`, `Unity Humanoid`, `Mixamo`, `Rigify`) rather than compact fantasy creatures.
3. **Post-Processing, Facial Rigging & GLB Export:** Headless Blender CLI (`blender --background --python script.py`) for manifold checks, decimation/retopology, UV/texture packing (matte PBR budget), adding/binding eye/eyelid/mouth bones or shape keys (`blink`, `curious`, `happy`), baking NLA clips, and validating `.glb` output.

## 4. Technical Trade-offs

| Stage | Capability | Verified Host Status (2026-11-17) | Limitation / Risk |
|---|---|---|---|
| 2D Concept & Turnaround (Stages 1–2) | Antigravity `image-generator` subagent (available in runtime) OR `gemini-api` CLI/SDK | `image-generator` subagent available; `gemini-api` CLI **NOT FOUND**; `GEMINI_API_KEY` **unset** | Multi-view turnarounds require careful consistency prompts/reference chaining (`image-generator` or Gemini Image reference fusion) |
| 3D Generation & Creature Auto-Rig (Stages 3, 5) | Tripo 3D API (`v3/animations/rig-check` + creature rigs) preferred over humanoid-centric Meshy | `TRIPO_API_KEY` **unset**; `MESHY_API_KEY` **unset** | Requires paid API account + credits for commercial private IP (Free tier does not grant unattributed commercial ownership); external riggers rarely produce facial blendshapes/eyelid bones |
| Automated Refinement, Facial Rig & GLB Export (Stages 4, 6, 7) | Headless Blender Python automation | `blender.exe` **NOT FOUND** on PATH or `C:\Program Files\Blender Foundation\` | Cannot run automated mesh cleanup, facial shape keys/eye rigging, or GLB export without installing Blender CLI locally |
| Runtime Integration (Stage 8) | Three.js + React Three Fiber + Drei (`useGLTF`, `useAnimations`) | Node `v24.19.0` present; packages deferred until Phase 0/5 | Must dispose GPU geometries/materials on unmount and pause `frameloop` when hidden/reduced-motion |

## 5. Recommended Decision

1. **Approve Pipeline Selection (Recommended):**
   - **Stages 1–2:** Use Antigravity `image-generator` (or Gemini Image API if configured) to create original concept sheets and front/3/4/side/back turnaround.
   - **Stage 3 & Base Rig (Stage 5a):** Prefer **Tripo 3D OpenAPI (Paid commercial tier)** over Meshy due to Tripo's non-humanoid creature rigging and `rig-check` endpoint, only after owner confirms commercial API credentials/credits.
   - **Stages 4, 5b, 6, 7:** Require **headless Blender CLI + Python scripts** (or equivalent verified offline Python/Blender pipeline) to verify topology, rig facial eyes/lids/mouth (since auto-riggers focus on body bones), generate missing expression/blink/look clips, optimize textures, export `mascot.glb`, and write `mascot_manifest.json`.
2. **Staged Preparation Gate (Approved Owner Plan):**
   - Headless Blender and 3D generation API services/credits are **intentionally deferred until entering Phase 5 (Mascot Production)** rather than blocking Phase 0–4 development.
   - When Phase 5 begins, verify: (a) local headless Blender CLI (`blender --version`) for automated mesh/facial-rig refinement, and (b) commercial 3D generation API credentials & credit budget (Tripo Paid tier or approved alternative) compatible with desktop software distribution (`THIRD_PARTY_NOTICES.md`).
3. **No Silent Substitution:** Never ship static SVG or fake 3D placeholders as the accepted mascot (`MASCOT_SPECIFICATION.md`).

## 6. Decision Status

**Approved** (Staged Phase 5 preparation plan; Tripo v3 + headless Blender pipeline **Recommended**) / **Blocked** (Phase 5 3D execution cannot run until Phase 5 prerequisites are installed and provisioned).

## 7. Consequences

- **Positive:** Prevents wasted Phase 5 effort on humanoid-only riggers (Meshy) or uncommercial Free-tier licenses; keeps Phase 0–4 unblocked while flagging exact Phase 5 prerequisites transparently.
- **Negative / Risk:** If paid 3D API credits are not provisioned by Phase 5, the AI agent must rely solely on headless Blender Python procedural modeling/rigging once Blender is installed, or treat Mascot integration as an explicit deferred release gate (`QA_TESTING_PLAN.md` §"Release gate").

## 8. Implementation Implications

- Phase 0–4 can implement the mascot slot layout, state machine hooks (`hidden`, `idle`, `greeting`, `curious`), reduced-motion guards, and graceful documented fallback state (`USER_FLOW.md` §8) without fabricating 3D assets.
- Keep `TRIPO_API_KEY` / `MESHY_API_KEY` strictly out of repo, logs, and manifests (`MASCOT_PRODUCTION_PIPELINE.md`).

## 9. Validation Requirements

1. Environment preflight check confirming `blender --version` and commercial API key/license terms before Phase 5 starts.
2. `mascot_manifest.json` + clean GLB load test in Three.js/R3F confirming all 10 clips play cleanly, `blink` / gaze clamps work, and render loop pauses at 0 frames/sec when hidden or reduced-motion is enabled.

## 10. Related Documents

- [`MASCOT_SPECIFICATION.md`](../MASCOT_SPECIFICATION.md)
- [`MASCOT_PRODUCTION_PIPELINE.md`](../MASCOT_PRODUCTION_PIPELINE.md)
- [`ANIMATION_SPECIFICATION.md`](../ANIMATION_SPECIFICATION.md)
- [`QA_TESTING_PLAN.md`](../QA_TESTING_PLAN.md)
- [`SKILL_UTILIZATION.md`](../SKILL_UTILIZATION.md)

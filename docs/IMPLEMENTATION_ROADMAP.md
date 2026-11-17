# Implementation Roadmap

## Delivery strategy

Design the whole product, implement in stages, and keep each stage testable. Do not build every screen before the tracking/data foundation is proven.

## Phase 0 — Repository and toolchain foundation

- Confirm documentation directory, establish Phase 0 ADRs (`docs/adr/ADR-001`..`008`), and align official branding (`Timo` — `A little clarity for your digital day.`).
- Discover and apply relevant installed Antigravity skills (`docs/SKILL_UTILIZATION.md`).
- Install missing host prerequisites (`rustc`/`cargo`, MSVC Build Tools, Context7 & Playwright MCPs) and create Tauri 2 + React + TypeScript project.
- Configure lint, format, Rust tests, frontend tests, and CI where available.
- Verify build on Windows 11.

**Exit gate:** clean project build, documented versions, owner confirmation on open Phase 0 policy items (`ADR-002`, `ADR-003`), no unverified skill claims.

## Phase 1 — Native tracking proof of concept

- Foreground event hook.
- Process identity resolution.
- Idle threshold.
- Lock/unlock and sleep/resume.
- Monotonic time accounting.
- Minimal SQLite session persistence.
- Console/diagnostic test harness.

**Exit gate:** critical tracking test matrix passes before full UI work.

## Phase 2 — Recognition and classification

- Executable metadata and package identity.
- Curated app rules.
- Steam/Epic manifests where safely accessible.
- Standalone EXE support.
- Category confidence/evidence.
- User correction and unknown review queue.

**Exit gate:** labeled recognition corpus and correction persistence pass.

## Phase 3 — Product UI MVP

- Onboarding and privacy introduction.
- Local Windows username suggestion with confirmation/edit/skip.
- Overview.
- Applications and Application Details.
- Timeline.
- Settings and data controls.
- Empty/loading/error states.

**Exit gate:** UI values reconcile against database test fixtures and live tracking.

## Phase 4 — Insights and design polish

- Weekly/monthly insights.
- Light/dark theme.
- Accessibility audit.
- Motion and micro-interactions.
- Visual QA at multiple window sizes and scaling levels.

**Exit gate:** accessibility, visual, and performance reviews completed.

## Phase 5 — Mascot production and integration

- Concept and multi-view design.
- 3D generation and refinement.
- Rig and animation clips.
- GLB optimization and manifest.
- R3F integration and pause behavior.

The mascot pipeline is owned by the AI agent. If external credits/API keys are required, document the blocker and continue feasible non-blocked tasks. Do not mark incomplete asset work as complete.

**Exit gate:** actual GLB, animation clips, preview, and runtime verification exist.

## Phase 6 — Hardening and distribution

- Database recovery/migrations.
- Performance benchmark.
- Security and privacy review.
- NSIS installer/uninstaller.
- Optional startup setting.
- Signed update pipeline only after secure signing setup.
- Release checklist and license notices.

**Exit gate:** QA release gates pass and release artifacts are verified.

## Future phases

Focus sessions, goals, break reminders, usage limits, media-aware activity, cross-device sync, and macOS/Linux should be considered only after v1 usage data and user feedback justify them.

# TODO — Timo

This list begins as a planning baseline. Check an item only after evidence exists in the repository or test output.

## Phase 0 — Product and repository foundation
- [x] Confirm repository location (`c:\TIMO\docs\`) and inspect existing files.
- [x] Create/verify a docs index (`docs/README.md`) and ADR folder (`docs/adr/ADR-001`..`008`, `docs/PHASE_0_DECISION_REPORT.md`).
- [x] Discover installed Antigravity skills and create/update `docs/SKILL_UTILIZATION.md`.
- [ ] Confirm open Phase 0 owner policy questions (`ADR-002` retroactive vs forward idle cutoff; `ADR-003` neutral `Others / Uncategorized` vs 7th `Web & Browsing` category).
- [ ] Install Rust (`rustc`/`cargo`) + Windows MSVC prerequisites and configure required MCPs (`Context7`, `Playwright`).
- [ ] Verify current stable package/tool versions through official docs/Context7.
- [ ] Create Tauri 2 + React + TypeScript skeleton on Windows 11.
- [ ] Add lint, formatting, test, and build scripts.

## Phase 1 — Tracking proof of concept
- [ ] Implement foreground event hook on a dedicated native thread.
- [ ] Resolve HWND, PID, process creation identity, and executable metadata.
- [ ] Implement bounded idle sampling and configurable threshold.
- [ ] Implement lock/unlock detection.
- [ ] Implement suspend/resume detection.
- [ ] Implement monotonic session timing and midnight handling.
- [ ] Implement SQLite migrations and session persistence.
- [ ] Pass background app exclusion tests.
- [ ] Pass rapid app switching and recovery tests.

## Phase 2 — Recognition and classification
- [ ] Implement application identity service.
- [ ] Read executable version metadata safely.
- [ ] Detect package identity/AUMID where available.
- [ ] Add curated application rules.
- [ ] Add Steam/Epic metadata support where safely available.
- [ ] Support standalone EXE and portable apps.
- [ ] Implement confidence/evidence state.
- [ ] Implement user overrides and unknown review queue.
- [ ] Build labeled recognition test corpus.

## Phase 3 — UI MVP
- [ ] Build Timo design tokens and core components.
- [ ] Build onboarding/privacy explanation.
- [ ] Implement local Windows username suggestion and editable greeting.
- [ ] Build Overview with real data and empty/error states.
- [ ] Build Timeline.
- [ ] Build Applications list and filters.
- [ ] Build Application Details.
- [ ] Build Settings and data controls.
- [ ] Implement system tray and hide/quit semantics.

## Phase 4 — Insights and polish
- [ ] Build weekly/monthly Insights.
- [ ] Add light/dark themes.
- [ ] Audit keyboard navigation and screen reader labels.
- [ ] Audit 100%, 125%, and 150% Windows scaling.
- [ ] Add reduced-motion support.
- [ ] Run visual regression review.

## Phase 5 — Mascot production
- [ ] Discover available image/3D/Blender skills and tools.
- [ ] Generate original mascot concepts and turnaround.
- [ ] Choose and document an approved concept.
- [ ] Generate 3D candidates with provenance.
- [ ] Refine geometry/materials automatically where possible.
- [ ] Rig the creature and validate deformation.
- [ ] Create required animation clips.
- [ ] Export optimized GLB and manifest.
- [ ] Integrate with React Three Fiber.
- [ ] Verify pause/cleanup/reduced-motion/performance behavior.

## Phase 6 — Release
- [ ] Run full QA and native tracking test matrix.
- [ ] Run performance benchmark and attach report.
- [ ] Complete privacy/security review.
- [ ] Verify database migration/recovery/export/delete.
- [ ] Build NSIS installer and test uninstall.
- [ ] Configure signing/updater only when credentials/process are ready.
- [ ] Create release notes and third-party notices.

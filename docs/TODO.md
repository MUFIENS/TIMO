# TODO — Timo

This list begins as a planning baseline. Check an item only after evidence exists in the repository or test output.

## Phase 0 — Product and repository foundation
- [x] Confirm repository location (`c:\TIMO\docs\`) and inspect existing files.
- [x] Create/verify a docs index (`docs/README.md`) and ADR folder (`docs/adr/ADR-001`..`008`, `docs/PHASE_0_DECISION_REPORT.md`).
- [x] Discover installed Antigravity skills and create/update `docs/SKILL_UTILIZATION.md`.
- [x] Confirm Phase 0 owner policy decisions (`ADR-002` forward idle cutoff, `ADR-003` 6-category neutral browser default, `ADR-006` documented Windows APIs priority, `ADR-007`/`ADR-008` staged prerequisites).
- [x] Prepare Phase 1 native prerequisites: install Rust stable (`rustc`), Cargo, MSVC Build Tools, and Windows SDK.
  - [x] Install and verify Rust stable (`rustc 1.100.0`), Cargo (`1.100.0`), `x86_64-pc-windows-msvc` target, `rustfmt`, and `clippy` via official `rustup` (`docs/ENVIRONMENT_AUDIT.md`).
  - [x] Verify Microsoft Edge WebView2 Runtime (`157.0.4341.46`), Node.js (`v24.19.0`), `npm` (`11.17.0`), and Git (`2.55.0.windows.4`).
  - [x] Verify user-installed Visual Studio Build Tools 2026 (`18.12.0`, MSVC `14.52.37025` `cl.exe`/`link.exe`) and Windows SDK (`10.0.26100.0`) (`docs/ENVIRONMENT_AUDIT.md`).
- [x] Configure Context7 MCP for documentation during development (GitHub MCP optional; Figma MCP remains excluded).
  - [x] Add workspace-scoped Antigravity Context7 plugin (`.agents/plugins/context7/plugin.json` & `mcp_config.json`) without modifying global MCP settings.
  - [x] Reload Antigravity MCP Servers and verify live `context7_context7` tools (`resolve-library-id` & `query-docs`) via `call_mcp_tool`.
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
- [ ] Configure Playwright MCP prior to UI testing (`ADR-008`).
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
- [ ] Prepare headless Blender CLI and external 3D generation service/credentials (`ADR-007`).
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

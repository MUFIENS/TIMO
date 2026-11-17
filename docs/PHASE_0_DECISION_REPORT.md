# Phase 0 Final Decision Report: Timo

**Product:** Timo  
**Official Tagline:** **A little clarity for your digital day.** (**APPROVED — OWNER DECISION**)  
**Phase:** Phase 0 — Specification Refinement & Architecture Decision Records (Final)  
**Date:** 2026-11-17  
**Repository:** `c:\TIMO` (documentation under `c:\TIMO\docs\`)

---

## 1. Phase 0 Overview

Phase 0 resolved all cross-document inconsistencies and technical ambiguities uncovered in `docs/PROJECT_COMPREHENSION_REPORT.md`, incorporated the owner's official branding decision and explicit architectural approvals (`ADR-002`, `ADR-003`, `ADR-006`, `ADR-007`, `ADR-008`), audited live host tooling, MCP integrations, and Antigravity skills, and synchronized all product, tracking, recognition, database, design, QA, and execution specifications prior to starting Phase 1.

---

## 2. Official Timo Brand Identity & Tagline Confirmation

| Attribute | Value | Status |
|---|---|---|
| Application Name | **Timo** | **Approved** (Owner Decision) |
| Official Tagline | **A little clarity for your digital day.** | **APPROVED — OWNER DECISION** |
| Superseded Tagline | *"Your time, understood."* | Removed from all active blueprint references |
| Design Identity | Soft Playful Neo-Brutalism | **Approved** (Owner Decision) |
| Brand Personality | Friendly, gentle, approachable, playful, calm, supportive, non-judgmental | **Approved** (Owner Decision) |

---

## 3. Seven Technical Findings Investigated & Resolved

1. **Finding 1 — Midnight Session Splitting (`ADR-001`):** Standardized on **canonical, unsplit UTC `usage_sessions`** (`started_at_utc`, `ended_at_utc`, monotonic `duration_ms`, capture-time `timezone_id` & `utc_offset_minutes`) with deterministic query-time local-day slicing across midnight crossings, 23h/25h DST days, and timezone changes.
2. **Finding 2 — Idle Detection Boundary (`ADR-002`, Owner Approved):** Adopted **forward cutoff at threshold confirmation** (`T_confirm`) with a 5-minute (`300s`) configurable default in Settings (`TR-010`). Time prior to confirmation stays counted as active foreground usage; idle starts strictly at `T_confirm`; both `last_input_at_utc` and `confirmed_at_utc` are persisted in `idle_intervals`; lock/sleep/resume clip open idle intervals; zero overlapping duration between active usage and idle intervals is enforced.
3. **Finding 3 — Default Browser Categorization (`ADR-003`, Owner Approved):** Retained the **6 existing MVP categories** (no `Web & Browsing` category added), recognized known browsers as `app_type = "browser"`, defaulted them neutrally to `Others / Uncategorized` without assuming Productivity or collecting URLs (`PV-004`), and persisted manual user overrides (`confirmed_user`).
4. **Finding 4 — Application Identity & Recovery (`ADR-006`, Documented API Priority Approved):** Prioritized **documented Windows APIs exclusively**: `OpenProcess(PROCESS_QUERY_LIMITED_INFORMATION)` + `QueryFullProcessImageNameW` + `GetProcessTimes`, `ApplicationFrameHost.exe` child unwrapping (`EnumChildWindows` + `GetPackageFamilyName` / `GetApplicationUserModelId`), and **`CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS)` / `WTSEnumerateProcessesW`** fallback on `ERROR_ACCESS_DENIED` (prohibiting `NtQuerySystemInformation` as a primary dependency). Added `stable_app_key` precedence, read-only PE `.rsrc`/icon PNG caching (`%LOCALAPPDATA%\Timo\icons\`), and 30-second WAL crash checkpoints (`last_checkpoint_utc`, `checkpoint_monotonic_ms`, `recovery_closed`).
5. **Finding 5 — Accessibility & Design Tokens (`ADR-004`):** Preserved Soft Playful Neo-Brutalism while fixing all measured WCAG 2.2 AA contrast failures (`#25232A` on pastels `>= 6.33:1`, `#5A46B5` solid/ink `7.05:1`, `#88828E` interactive border `>= 3.54:1`, `#29272B` chart/card stroke) and creating a full warm plum-charcoal Dark Theme (`#18171C`/`#222028`/`#2B2832`) script-verified via `antislop-human/contrast-check.py`.
6. **Finding 6 — WebView2 Memory & Native Edge Cases (`ADR-005`):** Decoupled Rust tracking from `WebviewWindow` lifetime via `RunEvent::ExitRequested` `prevent_exit()`, defined Close-to-Tray `Destroy & Recreate` vs `hide() + WebViewExtWindows::set_memory_usage_level(Low)` as the Phase 1/3 benchmark comparison pair on 8 GB Windows 11 hardware, and mandated R3F lazy-loading + `frameloop` pausing.
7. **Finding 7 — Staged Tooling & Mascot Dependencies (`ADR-007`, `ADR-008`, Owner Approved):** Audited local host environment (`Windows NT 10.0.26300.0`, 8 GB RAM, `node v24.19.0`, `python 3.14.7`, `git 2.55.0`) and formalized the owner-approved staged preparation plan across Phase 1, development, UI testing, and Phase 5.

---

## 4. Research Sources & Technical Evidence

- **WCAG 2.2 AA Contrast Verification:** Executed `C:\Users\asepm\.gemini\config\skills\antislop-human\contrast-check.py` against 48 Light and Dark token combinations (documented with exact ratios in `ADR-004` and `DESIGN_SYSTEM.md`).
- **Host Environment Verification:** Verified OS build `10.0.26300.0`, 8 GB RAM (`8,269,398,016` bytes), 8 logical processors, `node v24.19.0`, `python 3.14.7`, `git 2.55.0.windows.4`, and verified absence of `rustc`, `cargo`, `blender.exe`, `gemini-api` CLI, `playwright` CLI, `GEMINI_API_KEY`, `TRIPO_API_KEY`, `MESHY_API_KEY`, Context7 MCP, Playwright MCP, and GitHub MCP.
- **Microsoft Learn & Ecosystem Research (`search_web`):**
  - `CreateToolhelp32Snapshot` (`szExeFile`) and `WTSEnumerateProcessesW` (`pProcessName`) documented no-`OpenProcess` fallback behavior for elevated processes.
  - `ApplicationFrameHost.exe` child HWND resolution + `GetPackageFamilyName` / `GetApplicationUserModelId`.
  - Tauri v2 `ExitRequested` (`prevent_exit`) + `WebViewExtWindows::set_memory_usage_level(Low)` vs destroy/recreate lifecycle.
  - Tripo 3D `v3.0` (`v3/animations/rig-check`, non-humanoid creature rigs, Paid commercial ownership vs Free tier attribution) vs humanoid-focused Meshy API.

---

## 5. Final Decision Status Breakdown

### A. Approved Decisions (Owner Confirmed)
1. **Official Brand Identity:** Name **Timo**, official tagline **A little clarity for your digital day.** (**APPROVED — OWNER DECISION**), Soft Playful Neo-Brutalism design direction, friendly/gentle/non-judgmental voice.
2. **Platform & Architecture Baseline:** Windows 11 x64 first (Windows 10 best-effort only); Tauri 2 + Rust + React + TS + Vite + Tailwind + SQLite (Rust-owned); no login/cloud sync in v1; local Windows username greeting suggestion; foreground-only tracking excluding Timo by default; launcher vs game separation; 5 primary pages + nested `Application Details`; AI-owned 3D creature mascot (no manual Blender operation, no SVG shortcut).
3. **`ADR-002` — Idle Detection Policy:** Forward cutoff when the configurable 5-minute default threshold confirms (`T_confirm`); pre-threshold time stays active foreground usage; idle starts at `T_confirm`; persist both `last_input_at_utc` and `confirmed_at_utc`; zero overlap between active usage and `idle_intervals`.
4. **`ADR-003` — Browser Categorization:** Keep the 6 existing categories for MVP (no `Web & Browsing` category); recognize browsers as `app_type = "browser"`; default category `Others / Uncategorized`; never assume Productivity; persistent manual category override.
5. **`ADR-006` (API Constraint) — Documented Windows APIs Priority:** Prioritize documented Windows APIs (`OpenProcess(PROCESS_QUERY_LIMITED_INFORMATION)`, `GetPackageFamilyName`, `CreateToolhelp32Snapshot`, `WTSEnumerateProcessesW`); `NtQuerySystemInformation` must NOT be a primary dependency.
6. **`ADR-007` & `ADR-008` — Staged Prerequisites & MCP Policy:**
   - Rust stable, Cargo, MSVC Build Tools, Windows SDK before Phase 1.
   - Context7 MCP during development.
   - Playwright MCP before UI testing (Phase 3/4).
   - GitHub MCP optional.
   - Headless Blender + 3D generation service when entering Phase 5 mascot production.
   - Figma MCP permanently excluded.

### B. Provisional Decisions (Pending Implementation Benchmarks / Visual QA)
1. **`ADR-004` — Light & Dark Semantic Theme Tokens:** Contrast ratios verified mathematically against WCAG 2.2 AA (`4.5:1` / `3.0:1`), provisional pending visual verification on Windows 11 at `100%`, `125%`, and `150%` DPI scaling (`Phase 3/4`).
2. **`ADR-005` — Tray Memory Strategy & Performance Budgets:** `Destroy & Recreate` vs `hide() + MemoryUsageLevel::Low` for Close-to-Tray, and `<= 100 MB` tray RAM / `<= 150 MB` dashboard RAM / ~0% idle CPU targets remain provisional until release-build benchmarking (`Phase 1/3/6`).
3. **Engine & UI Tuning Defaults:** `75–150 ms` foreground focus debounce, ~1 Hz `GetLastInputInfo` sampling interval, `980x640` minimum logical window target, and animation timing bands (`120–500 ms`).
4. **Open Analytical Formula Detail (`G-03`):** Exact denominator rule for `Average daily use` (`UI-005`, e.g., calendar days in selected range vs active days) and sub-minute rounding display format to be finalized cleanly with SQL query specs prior to Phase 3.

### C. Recommended Decisions (Engineering Baseline Ready for Validation)
1. **`ADR-001` — Midnight Handling:** Canonical unsplit UTC `usage_sessions` + monotonic `duration_ms` + deterministic query-time local calendar & DST slicing.
2. **`ADR-006` — Identity & Recovery Mechanisms:** `stable_app_key` precedence (`pkg:` > `meta:`/`steam:`/`epic:` > `exe:`), read-only PE `.rsrc`/icon PNG caching in `%LOCALAPPDATA%\Timo\icons\` (purged on delete), and 30-second WAL recovery checkpoints (`last_checkpoint_utc`, `checkpoint_monotonic_ms`, `recovery_closed`).
3. **`ADR-007` — Mascot Pipeline Selection:** Tripo 3D Paid commercial pipeline (`rig-check` + creature rigs) + headless Blender CLI for facial expressions/eyelids + GLB export (to be provisioned in Phase 5).
4. **Custom Timeline & Evergreen Baseline:** Custom CSS Grid/SVG `UsageTimeline` (`TECHSTACK.md`), Windows 11 Evergreen WebView2 CSS/DOM baseline, and loopback WebView2 CDP inspection (`ADR-008`).

### D. Blocked Items & Required Staged Preparations

| Stage | Required Preparation | Current Status |
|---|---|---|
| **Before Phase 1** | Install **Rust stable (`rustup`)**, **Cargo**, **MSVC Build Tools**, and **Windows 11 SDK** | **Required / Not yet installed** (`rustc` & `cargo` absent) |
| **During Dev (Phase 0/1+)** | Configure **Context7 MCP** (and keep **GitHub MCP** optional, **Figma MCP** excluded) | **Pending setup** (not in active `<mcp_servers>`) |
| **Before UI Testing (Phase 3/4)** | Configure **Playwright MCP** (paired with loopback WebView2 CDP & native tests) | **Pending setup** (deferred until UI testing per staged plan) |
| **Before Mascot Production (Phase 5)** | Install **headless Blender CLI** and provision **commercial 3D generation API/credits** (`Tripo` Paid tier or alternative) | **Blocked until Phase 5** (`blender.exe` & API keys absent) |
| **Release Signing (Phase 6 / Post-v1)** | Code-signing certificate & updater key infrastructure | **Blocked** (unsigned NSIS dev/test builds first; updater disabled until signing ready) |

### E. Questions Requiring Owner Approval
**None remaining for Phase 0.** All owner policy questions (`ADR-002`, `ADR-003`, `ADR-006` API priority, `ADR-007`/`ADR-008` staged prerequisites, and official tagline) have been explicitly resolved.

---

## 6. ADRs & Updated Documents Summary

- **ADRs Created ([`docs/adr/`](./adr/)):** `ADR-001` through `ADR-008`.
- **Specifications Synchronized:** `README.md`, `PRD.md`, `PRODUCT_VISION.md`, `PRODUCT_REQUIREMENTS.md`, `DESIGN.md`, `DESIGN_SYSTEM.md`, `STYLEGUIDE.md`, `TRACKING_ENGINE_SPEC.md`, `APPLICATION_RECOGNITION_SPEC.md`, `DATABASE_SCHEMA.md`, `SYSTEM_ARCHITECTURE.md`, `ARCHITECTURE.md`, `PRIVACY_SECURITY.md`, `SECURITY.md`, `PERFORMANCE_REQUIREMENTS.md`, `MASCOT_PRODUCTION_PIPELINE.md`, `MCP_CONFIGURATION.md`, `QA_TESTING_PLAN.md`, `AGENTS.md`, `IMPLEMENTATION_ROADMAP.md`, `TODO.md`, `PROJECT_COMPREHENSION_REPORT.md`, `SKILL_UTILIZATION.md`, and `PHASE_0_DECISION_REPORT.md`.

---

## 7. Skills & MCP Summary

- **Skills Applied:** Logged in [`docs/SKILL_UTILIZATION.md`](./SKILL_UTILIZATION.md) (`antislop`, `antislop-human` + `contrast-check.py`, `antislop-ui`, `antislop-copywriting`, `antislop-layoutmobile`, `design-system`, `ui-styling`, `ui-animation`, `motion-design`, `impeccable`, `ui-ux-pro-max`, `frontend-design`, `gemini-api-dev`, plus read-only checks on `design`, `brand`, `awesome-design`, `web-design-guidelines`, `vercel-composition-patterns`, `modern-web-guidance`, `a11y-debugging`, `memory-leak-debugging`, `antislop-code`, `gemini-api-cli`). Missing skill domains (Rust, Tauri 2, Windows Win32, SQLite, automated testing, Blender/3D) remain logged honestly.
- **MCPs:** `shadcn`, `magicuidesign-mcp`, `gemini-api_gemini-api-docs`, and `data-agent-kit-plugin_*` active in `<mcp_servers>` (unused in docs-only phase); `Context7`, `Playwright MCP`, and `GitHub MCP` not currently connected; `Figma MCP` excluded.

---

## 8. Readiness Assessment for Phase 1

- **Documentation & Architecture Readiness:** **READY.** Specifications, DB schema, acceptance criteria (`TR-005`, `AR-005`, `AR-007`, `QA-004`, `QA-010`), ADRs, and theme contrast baselines are aligned with zero unresolved policy contradictions.
- **Execution Readiness for Phase 1:** **Awaiting Prerequisite Setup & Explicit Owner Go-Ahead.** Once Rust stable, Cargo, MSVC Build Tools, Windows SDK (and Context7 MCP as desired) are ready on the workstation and you confirm starting Phase 1, work can proceed cleanly to Tauri 2 + Rust scaffolding and the native Windows tracking PoC.

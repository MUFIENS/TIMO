# Phase 0 Decision Report: Timo

**Product:** Timo  
**Official Tagline:** **A little clarity for your digital day.** (**APPROVED — OWNER DECISION**)  
**Phase:** Phase 0 — Specification Refinement & Architecture Decision Records  
**Date:** 2026-11-17  
**Repository:** `c:\TIMO` (documentation under `c:\TIMO\docs\`)

---

## 1. Phase 0 Overview

Phase 0 resolved the branding update and seven technical gaps identified during full blueprint immersion (`docs/PROJECT_COMPREHENSION_REPORT.md`), established eight formal Architecture Decision Records (`docs/adr/ADR-001` through `ADR-008`), audited live host tooling, MCP servers, and Antigravity skill availability, and synchronized all affected product, engineering, and design specifications without starting code implementation.

---

## 2. Official Timo Brand Identity & Tagline Confirmation

| Attribute | Value | Status |
|---|---|---|
| Application Name | **Timo** | **Approved** (Owner Decision) |
| Official Tagline | **A little clarity for your digital day.** | **APPROVED — OWNER DECISION** |
| Superseded Tagline | *"Your time, understood."* | Removed from all active blueprint references |
| Design Identity | Soft Playful Neo-Brutalism | **Approved** (Owner Decision) |
| Brand Personality | Friendly, gentle, approachable, playful, calm, supportive, non-judgmental | **Approved** (Owner Decision) |

All relevant references across `README.md`, `PRD.md`, `PRODUCT_VISION.md`, `PRODUCT_REQUIREMENTS.md`, `DESIGN.md`, `DESIGN_SYSTEM.md`, `STYLEGUIDE.md`, `AGENTS.md`, `IMPLEMENTATION_ROADMAP.md`, and `PROJECT_COMPREHENSION_REPORT.md` have been updated to **A little clarity for your digital day.**

---

## 3. Seven Technical Findings Investigated

1. **Finding 1 — Midnight Session Splitting (`ADR-001`):** Resolved open contradiction between write-time splitting vs query-time slicing by standardizing on **canonical unsplit UTC `usage_sessions`** with strictly monotonic `duration_ms`, capture-time `timezone_id` / `utc_offset_minutes`, and deterministic query-time local-day slicing handling midnight crossings, 23h/25h DST days, and timezone travel.
2. **Finding 2 — Idle Detection Boundary (`ADR-002`):** Evaluated retroactive cutoff at `last_input_time` vs forward cutoff at `threshold_crossed_at`. Recommended retroactive closure clamped safely to `max(last_input_time, current_session_started_at)` once the 5-minute default threshold is confirmed, paired with 32-bit `GetTickCount()` wraparound math (`wrapping_sub`), a dedicated `idle_intervals` table, lock/sleep clipping while idle, and explicit passive-media disclosure.
3. **Finding 3 — Default Browser Categorization (`ADR-003`):** Resolved browser policy consistently across `PRD.md`, `APPLICATION_RECOGNITION_SPEC.md`, `PRODUCT_REQUIREMENTS.md`, and `DATABASE_SCHEMA.md`: recognize browsers accurately (`app_type = "browser"`), default category neutrally to `Others / Uncategorized` with `classification_state = "low"` (category certainty) so they appear in the Review Queue without assuming `Productivity` or tracking URLs (`PV-004`).
4. **Finding 4 — Application Identity & Recovery (`ADR-006`):** Defined UWP `ApplicationFrameHost.exe` child HWND unwrapping (`EnumChildWindows` + `GetPackageFamilyName` / `GetApplicationUserModelId`), `ERROR_ACCESS_DENIED` elevated/anti-cheat fallback (`NtQuerySystemInformation(SystemProcessIdInformation)` + `Toolhelp32`), deterministic `stable_app_key` hierarchy protecting portable EXEs, read-only PE `.rsrc`/icon extraction into local PNG cache (`%LOCALAPPDATA%\Timo\icons\`, deleted on purge), and 30-second WAL recovery checkpoints (`last_checkpoint_utc`, `checkpoint_monotonic_ms`, `recovery_closed`).
5. **Finding 5 — Accessibility & Design Tokens (`ADR-004`):** Preserved Soft Playful Neo-Brutalism while resolving every measured WCAG 2.2 AA contrast failure (`#FFFFFF` on `#A99AEF` 2.45:1 and `#F38B78` 2.39:1; `#E5E1D9` and borderless pastel bars < 3:1), pairing pastel fills with `#25232A` (`6.33:1`–`11.49:1`), introducing `#5A46B5` solid/ink primary (`7.05:1`), `#88828E` interactive border (`3.54–3.73:1`), and creating a complete warm plum-charcoal Dark Theme (`#18171C` / `#222028` / `#2B2832`) with all text/accent/focus-ring pairs script-verified (`>= 4.5:1` text, `>= 3.0:1` non-text).
6. **Finding 6 — WebView2 Memory & Windows Native Edge Cases (`ADR-005`):** Decoupled the Rust tracking engine from `WebviewWindow` lifetime via `RunEvent::ExitRequested` `prevent_exit()`, specified Close-to-Tray `Destroy & Recreate` vs `hide() + WebViewExtWindows::set_memory_usage_level(Low)` as the Phase 1/3 benchmark test pair against the `<= 100 MB` provisional tray budget on 8 GB Windows 11 hardware, and enforced R3F lazy-loading + `frameloop` pause rules.
7. **Finding 7 — Development Tooling & Mascot Dependencies (`ADR-007`, `ADR-008`):** Audited local Windows 11 environment (`Windows NT 10.0.26300.0`, 8 GB RAM, 8 logical processors), verifying present tools (`git 2.55.0`, `node v24.19.0`, `npm`, `python 3.14.7`) and missing prerequisites (`rustc`, `cargo`, `blender.exe`, `gemini-api` CLI, `playwright` CLI, `TRIPO_API_KEY`, `MESHY_API_KEY`, `GEMINI_API_KEY`, Context7 MCP, Playwright MCP, GitHub MCP).

---

## 4. Research Sources & Technical Evidence

- **WCAG 2.2 AA Contrast Verification:** Executed `C:\Users\asepm\.gemini\config\skills\antislop-human\contrast-check.py` directly on 48 Light and Dark token combinations (full ratios recorded in `ADR-004` and `DESIGN_SYSTEM.md`).
- **Host Environment Verification:** PowerShell tool/env checks on `c:\TIMO` verifying OS build `10.0.26300.0`, `8,269,398,016` bytes physical RAM, `node v24.19.0`, `python 3.14.7`, `git 2.55.0.windows.4`, and missing `rustc`/`cargo`/`blender.exe`/API env keys.
- **Official Web Technical Research (`search_web`):**
  - Tauri v2 tray lifecycle (`RunEvent::ExitRequested` + `api.prevent_exit()`, `WindowEvent::CloseRequested` + `api.prevent_close()`) and Windows `WebViewExtWindows::set_memory_usage_level(MemoryUsageLevel::Low)` vs destroy/recreate WebView2 memory trade-offs.
  - Windows `ApplicationFrameHost.exe` child HWND enumeration (`EnumChildWindows`, `PROCESS_QUERY_LIMITED_INFORMATION`, `GetPackageFamilyName`, `GetApplicationUserModelId`), `GetLastInputInfo` 32-bit `GetTickCount()` 49.7-day wraparound math, and `NtQuerySystemInformation(SystemProcessIdInformation, 0x58)` / `CreateToolhelp32Snapshot` fallback for `ERROR_ACCESS_DENIED` elevated/anti-cheat processes.
  - Tripo 3D API (`v3.0`, `v3/animations/rig-check`, non-humanoid creature rigs, Free attribution/Tripo IP vs Paid private commercial ownership, ~25 credits/rig + ~10-20 credits/clip) vs Meshy API (humanoid/bipedal focus).

---

## 5. Decision Classification Matrix

### Approved Decisions
- Product Name: **Timo**; Official Tagline: **A little clarity for your digital day.** (**APPROVED — OWNER DECISION**).
- Design Identity: Soft Playful Neo-Brutalism; Windows 11 x64 primary (Windows 10 best-effort only); local-first SQLite owned by Rust; no login/cloud sync in v1; foreground-only tracking excluding Timo by default; launcher vs game separation; 5 primary pages + nested `Application Details`; local Windows username onboarding suggestion; AI-owned 3D creature mascot without manual Blender operation or static SVG substitution; approved technology stack (`ADR-008` policy); Figma MCP permanently excluded.

### Provisional Decisions
- **`ADR-002`:** 5-minute (`300`s) default idle threshold with retroactive session closure clamped to `max(last_input_time, session_start)` (`Requires Owner Confirmation`).
- **`ADR-004`:** Refined Light Theme and newly authored Dark Theme semantic color tokens (contrast math verified; pending 100%/125%/150% Windows DPI visual audit).
- **`ADR-005`:** WebView2 tray window lifecycle (`Destroy & Recreate` vs `hide() + MemoryUsageLevel::Low`), `<= 100 MB` tray RAM, `<= 150 MB` dashboard RAM, ~0% idle CPU targets (unbenchmarked until Phase 1/3).
- Debounce window (`75-150 ms`) and idle check cadence (~1 Hz).

### Recommendations
- **`ADR-001`:** Canonical unsplit UTC `usage_sessions` + monotonic `duration_ms` + query-time local calendar segmentation.
- **`ADR-003`:** Neutral browser default (`Others / Uncategorized`, `app_type = "browser"`, `classification_state = "low"` for Review Queue) (`Requires Owner Confirmation`).
- **`ADR-006`:** UWP `ApplicationFrameHost.exe` child unwrapping, `SystemProcessIdInformation`/`Toolhelp32` elevated fallback, `stable_app_key` precedence, read-only PE `.rsrc` PNG icon cache, and 30s WAL crash checkpoints.
- **`ADR-007` (Toolchain Choice):** Tripo v3 non-humanoid pipeline (Paid commercial plan) + headless Blender CLI facial rigging + GLB export.
- **`ADR-008` (Browser Baseline):** Windows 11 Evergreen WebView2 CSS/DOM baseline + WebView2 loop-back CDP inspection alongside Playwright.

### Blocked Items
- **Phase 0 Compilation / Scaffolding:** Blocked on missing local `rustc`/`cargo` (and MSVC Build Tools verification).
- **MCP Workflows (`ADR-008`):** `Context7` and `Playwright MCP` are missing from active `<mcp_servers>` (Context7 needed for pinned crate/package lookup; Playwright MCP needed for browser UI QA).
- **Phase 5 Mascot 3D Execution (`ADR-007`):** Blocked on missing local `blender.exe`, unset `TRIPO_API_KEY`/`MESHY_API_KEY`/`GEMINI_API_KEY`, and unprovisioned paid 3D commercial credits.
- **Release Signing & Auto-Updater:** Blocked on code-signing certificate and key infrastructure (post-v1 / release phase).

### Questions Requiring Owner Confirmation
1. **`ADR-002` (Idle Cutoff Policy):** Confirm whether retroactively trimming the active app session back to `last_input_time` when 5 minutes of inactivity is reached (avoiding +5m break inflation, at the cost of a "cliff" when passively reading/watching past 5m) matches your intended UX, or if you prefer forward cutoff at threshold crossing (`+5m`).
2. **`ADR-003` (Default Browser Category):** Confirm whether known browsers (`Chrome`, `Edge`, `Firefox`, etc.) should default to **`Others / Uncategorized`** (preserving the exact 6 approved categories) or whether you want to approve a 7th dedicated **`Web & Browsing`** category.
3. **`ADR-007` / `ADR-008` (Tooling Provisioning):** Confirm whether you plan to configure `Context7` + `Playwright` MCPs and install Rust (`rustup`) + Blender CLI locally (and provide commercial Tripo API access/credits later for Phase 5, or prefer pure procedural Blender Python generation when Blender is installed).

---

## 6. Architecture Decision Records Created

- [`docs/adr/ADR-001-midnight-session-handling.md`](./adr/ADR-001-midnight-session-handling.md)
- [`docs/adr/ADR-002-idle-detection-policy.md`](./adr/ADR-002-idle-detection-policy.md)
- [`docs/adr/ADR-003-application-category-defaults.md`](./adr/ADR-003-application-category-defaults.md)
- [`docs/adr/ADR-004-accessibility-and-theme-tokens.md`](./adr/ADR-004-accessibility-and-theme-tokens.md)
- [`docs/adr/ADR-005-background-memory-strategy.md`](./adr/ADR-005-background-memory-strategy.md)
- [`docs/adr/ADR-006-application-identity-and-recovery.md`](./adr/ADR-006-application-identity-and-recovery.md)
- [`docs/adr/ADR-007-mascot-production-toolchain.md`](./adr/ADR-007-mascot-production-toolchain.md)
- [`docs/adr/ADR-008-development-tooling-and-mcp.md`](./adr/ADR-008-development-tooling-and-mcp.md)

---

## 7. Documents Updated

- Branding & product definition: `README.md`, `PRD.md`, `PRODUCT_VISION.md`, `PRODUCT_REQUIREMENTS.md`, `DESIGN.md`, `DESIGN_SYSTEM.md`, `STYLEGUIDE.md`, `AGENTS.md`.
- Engineering, architecture & schemas: `TRACKING_ENGINE_SPEC.md`, `APPLICATION_RECOGNITION_SPEC.md`, `DATABASE_SCHEMA.md`, `SYSTEM_ARCHITECTURE.md`, `ARCHITECTURE.md`, `PRIVACY_SECURITY.md`, `SECURITY.md`, `PERFORMANCE_REQUIREMENTS.md`.
- Tooling, mascot & roadmap: `MASCOT_PRODUCTION_PIPELINE.md`, `MCP_CONFIGURATION.md`, `IMPLEMENTATION_ROADMAP.md`, `TODO.md` (checked only pure Phase 0 documentation/skill discovery items; all code/toolchain/build items remain `[ ]`), `PROJECT_COMPREHENSION_REPORT.md`, and `SKILL_UTILIZATION.md`.

---

## 8. Skills Actually Used

Recorded with exact file paths, read status, applied methods, and outputs in [`docs/SKILL_UTILIZATION.md`](./SKILL_UTILIZATION.md):
- `antislop`, `antislop-human` (executed `contrast-check.py` on 48 Light/Dark pairs + read `WCAG.md`), `antislop-ui`, `antislop-copywriting`, `antislop-layoutmobile`
- `design-system` (`token-architecture.md`, `semantic-tokens.md`, `states-and-variants.md`)
- `ui-styling` (`shadcn-theming.md`), `ui-animation` (`decision-framework.md`, `performance-deep-dive.md`), `motion-design` (`context-adaptation.md`)
- `impeccable` (`operate.md`, `audit.md`), `ui-ux-pro-max` (`quick-reference.md`, `pro-rules.md`), `frontend-design`, `gemini-api-dev`

Missing skill domains (Rust, Tauri 2, Windows Win32/WinRT, SQLite, automated testing, Blender/3D rigging) remain honestly documented in `SKILL_UTILIZATION.md`.

---

## 9. MCP Availability

- **Present in session `<mcp_servers>` (unused in this docs-only phase):** `shadcn`, `magicuidesign-mcp`, `gemini-api_gemini-api-docs`, `data-agent-kit-plugin_dak-data-agent-kit`, `data-agent-kit-plugin_dak-notebook`.
- **Missing (Required by `MCP_CONFIGURATION.md`):** `Context7`, `Playwright MCP`.
- **Missing (Optional / Excluded):** `GitHub MCP` (absent), `Figma MCP` (excluded per owner rule).

---

## 10. Mascot Production Dependencies

- **2D Concept Art (Stages 1–2):** `image-generator` subagent available in runtime (`gemini-api` CLI missing; `GEMINI_API_KEY` unset).
- **3D Generation & Body Rigging (Stages 3, 5a):** Tripo 3D Paid commercial API key + credits needed (`TRIPO_API_KEY` unset; Free tier incompatible with commercial unattributed ownership).
- **Refinement, Facial Rigging & GLB Export (Stages 4, 5b, 6, 7):** Headless Blender CLI required (`blender.exe` not currently installed).

---

## 11. Unresolved Owner Decisions

1. Idle retroactive trim to `last_input_time` vs forward cutoff (`ADR-002`).
2. Browser default under `Others / Uncategorized` vs 7th `Web & Browsing` category (`ADR-003`).
3. Provisioning strategy for Rust/Blender local tools, Context7/Playwright MCPs, and Phase 5 commercial 3D generation credits (`ADR-007`, `ADR-008`).

---

## 12. Remaining Implementation Blockers

- Missing `rustc` / `cargo` (and MSVC Build Tools verification) prevents Tauri 2 workspace initialization and Phase 1 PoC compilation.
- Missing `Context7` MCP blocks automated crate/npm version lookup inside Antigravity until installed or fallback official-doc verification is approved.
- Missing `Playwright MCP` blocks MCP-driven browser E2E checks in Phase 3/4.
- Missing `blender.exe` + paid `TRIPO_API_KEY` blocks automated Phase 5 3D mascot production.

---

## 13. QA & Validation Requirements Before Moving Past Phase 1

- All `ADR-001`, `ADR-002`, and `ADR-006` invariants covered by Rust unit & Windows 11 native integration tests (`QA-001`..`QA-014`).
- Wraparound-safe `GetLastInputInfo` math, `ApplicationFrameHost.exe` UWP unwrapping, `ERROR_ACCESS_DENIED` elevated/anti-cheat fallback, and `TerminateProcess` crash checkpoint recovery tested natively on Windows 11.
- Early memory comparison of `Destroy & Recreate` vs `hide() + MemoryUsageLevel::Low` recorded across 3 release-build trials on the 8 GB Windows 11 host (`ADR-005`).
- Full `DESIGN_SYSTEM.md` Light & Dark visual audit across `100%`, `125%`, and `150%` DPI scaling and `<MotionConfig reducedMotion="user">` verification (`ADR-004`, `QA-015`).

---

## 14. Readiness Assessment for Phase 1

- **Specification Readiness:** **READY** (pending owner confirmation on `ADR-002` idle cutoff and `ADR-003` browser category preference). All architectural contradictions, database schema omissions, WCAG AA contrast defects, and Windows tracking edge cases have documented designs.
- **Environment Readiness:** **BLOCKED FOR CODE EXECUTION** until Rust (`rustc`/`cargo`) and Windows MSVC Build Tools are installed on the host. Once installed and Phase 0 scaffolding completes, Phase 1 (the native Windows tracking PoC) can begin immediately.

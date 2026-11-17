# ADR-008: Development Toolchain, Staged Prerequisites & MCP Availability

**Date:** 2026-11-17  
**Status:** Approved (Owner approved staged preparation sequence & Figma MCP exclusion) / Blocked (Phase 1 compilation requires installing Rust/Cargo/MSVC/Windows SDK; Context7 & Playwright MCPs pending setup per stage)  
**Owners:** Release Engineering, AI Agent Operations  

---

## 1. Context

`TECH_STACK.md`, `MCP_CONFIGURATION.md`, `WORKFLOW.md`, and `QA_TESTING_PLAN.md` mandate verifying stable package versions via Context7 and official documentation, building with Tauri 2 + Rust + Vite/React on Windows 11 x64, auditing UI via Playwright MCP, and excluding Figma MCP.

## 2. Problem

Phase 0 audit on 2026-11-17 revealed a gap between required blueprint tooling and what is currently installed or connected on the host and inside the Antigravity MCP session. Proceeding into Phase 1/3 without documenting exact gaps would violate `SKILL.md`, `AGENTS.md`, and `GEMINI.md` anti-fabrication rules.

## 3. Options Considered

1. **Ignore missing MCPs/Rust toolchain and claim Phase 0 readiness.** (Rejected: violates anti-fabrication policy.)
2. **Document exact verified environment inventory, align WebView2 CDP/Playwright fallbacks, and gate Phase 0 Tauri scaffolding + Phase 3 UI checks on verified prerequisites.**

## 4. Technical Trade-offs

### Verified Host Tooling Inventory (2026-11-17)
- **OS & Reference Hardware:** Windows 11 (`Microsoft Windows NT 10.0.26300.0`), ~8 GB RAM (`8,269,398,016` bytes), 8 logical processors — matches the 8 GB Windows 11 baseline test profile in `PERFORMANCE_REQUIREMENTS.md`.
- **Available Locally:** `git 2.55.0.windows.4`, `node v24.19.0`, `npm`, `python 3.14.7`.
- **Missing Locally (Required for Phase 0/1/5):**
  - `rustc` / `cargo` (**NOT FOUND** on PATH or `%USERPROFILE%\.cargo\bin`) — required for Tauri 2 & Windows tracking core (alongside Visual Studio C++ Build Tools / Windows SDK & WebView2 Runtime).
  - `tauri` CLI, `pnpm` (optional), `playwright` CLI, `blender.exe` (**NOT FOUND**).

### Verified MCP Server Inventory (2026-11-17)
| Integration | Blueprint Requirement | Current Session Status | Operational Impact |
|---|---|---|---|
| **Context7 MCP** | Required (`MCP_CONFIGURATION.md`) | **Missing** (not in active `<mcp_servers>`) | Live crate/npm doc lookup via Context7 unavailable; fallback is official documentation search (`search_web`) until configured |
| **Playwright MCP** | Required (`MCP_CONFIGURATION.md`) | **Missing** (not in active `<mcp_servers>`) | Automated browser/DOM accessibility & layout assertions via Playwright MCP unavailable until configured |
| **GitHub MCP** | Optional (`MCP_CONFIGURATION.md`) | **Missing** | Non-blocking; local `git` CLI is installed |
| **Figma MCP** | **Explicitly Excluded** | **Absent (Compliant)** | Must remain excluded |
| `shadcn` MCP | Not in blueprint list | **Active in `<mcp_servers>`** | Useful in Phase 3 for verifying shadcn/ui registry items & Tailwind v4 integration |
| `magicuidesign-mcp`, `gemini-api_gemini-api-docs`, `data-agent-kit-plugin_*` | Not in blueprint list | **Active in `<mcp_servers>`** | `gemini-api-docs` relevant if writing Gemini API code for mascot concepts; DAK unrelated to Timo |

### Verified Antigravity Skill Inventory Gap (`SKILL.md`)
- Installed and applicable skills exist for UI/UX, design tokens, WCAG contrast, React composition, motion, and Chrome DevTools debugging (`SKILL_UTILIZATION.md`).
- **Missing installed skills:** Rust, Tauri 2, Windows Win32/WinRT native APIs, embedded SQLite, Playwright/automated testing, and Blender/3D modeling/rigging.

## 5. Recommended Decision

Adopt the **Owner-Approved Staged Preparation Plan**:
1. **Before Phase 1 (Mandatory Native Prerequisites):**
   - Install **Rust stable (`rustup`)**, **Cargo**, **Microsoft Visual Studio C++ Build Tools**, and **Windows 11 SDK**, and confirm Edge WebView2 Evergreen Runtime on the Windows 11 host before creating the Tauri skeleton and running `cargo test`.
2. **During Development (Phase 0/1+):**
   - Configure **Context7 MCP** to verify up-to-date Tauri 2, Rust crate, React, Tailwind, Motion, Recharts, and Three.js/R3F APIs during implementation.
3. **Before UI Testing (Phase 3/4):**
   - Configure **Playwright MCP** before running browser/DOM accessibility and layout verification (paired with Windows loopback WebView2 CDP `--remote-debugging-port=9222` per `impeccable` `audit.md`, alongside native Windows integration tests).
4. **Optional Integration:**
   - **GitHub MCP** remains optional (local `git` CLI is already installed).
5. **Phase 5 Mascot Stage (`ADR-007`):**
   - Prepare **headless Blender** and a **3D generation service** only when entering Phase 5 Mascot production.
6. **Permanent Exclusion:**
   - **Figma MCP remains excluded** and must not be added or enabled.
7. **Browser / WebView2 Policy (`modern-web-guidance`):**
   - Document in `AGENTS.md` that Timo targets **Windows 11 Evergreen Edge WebView2 (Chromium)**, allowing modern Baseline Widely Available CSS/DOM features without legacy browser polyfills.

## 6. Decision Status

**Approved** (Owner-approved staged preparation sequence, optional GitHub MCP, and permanent Figma MCP exclusion) / **Blocked** (Phase 1 Rust/Tauri compilation blocked until Rust stable, Cargo, MSVC Build Tools, and Windows SDK are installed; Context7 & Playwright MCPs pending setup in their respective stages).

## 7. Consequences

- **Positive:** Prevents silent version drift or blocked build surprises when Phase 0/1 code generation is authorized.
- **Negative / Risk:** Requires host environment setup (Rust + MSVC + MCP configuration) before executing `cargo test` or Playwright E2E scripts.

## 8. Implementation Implications

- Do not mark Phase 0 toolchain/skeleton items complete in `TODO.md` until `rustc --version`, `cargo --version`, and `npm run tauri build`/`cargo check` actually execute and pass.
- Update `MCP_CONFIGURATION.md` and `AGENTS.md` to reference WebView2 Evergreen baseline and WebView2 CDP inspection alongside Playwright MCP.

## 9. Validation Requirements

1. `rustc --version`, `cargo --version`, `node --version`, and Windows SDK verification logged at Phase 0 scaffolding start.
2. Context7 and Playwright MCP smoke checks (or explicit documented fallback verification) before Phase 1/3.

## 10. Related Documents

- [`MCP_CONFIGURATION.md`](../MCP_CONFIGURATION.md)
- [`TECH_STACK.md`](../TECH_STACK.md)
- [`TECHSTACK.md`](../TECHSTACK.md)
- [`SKILL_UTILIZATION.md`](../SKILL_UTILIZATION.md)
- [`WORKFLOW.md`](../WORKFLOW.md)
- [`TODO.md`](../TODO.md)

# AGENTS.md — Instructions for All Coding Agents

## Mission

Build Timo according to the approved Product Blueprint. Prioritize tracking correctness, privacy, reliability, accessibility, and measured performance over decorative UI.

## Mandatory first steps

1. Read `README.md`, `PRD.md`, `PRODUCT_REQUIREMENTS.md`, `SYSTEM_ARCHITECTURE.md`, and `WORKFLOW.md`.
2. Read the feature-specific specification before modifying that feature.
3. Follow `SKILL.md`: discover installed Antigravity skills, read the full relevant instructions/resources, apply them, and document actual use.
4. Inspect the existing repository before proposing file changes. Do not assume it is empty or overwrite existing work.
5. Verify current API/library documentation with Context7 and official sources when available.

## Non-negotiable product decisions

- Product name: Timo. Official tagline: **A little clarity for your digital day.** (**APPROVED — OWNER DECISION**).
- Windows 11 x64 first; Windows 10 compatibility is best-effort, not a security-supported promise.
- Browser/WebView baseline: Windows 11 Evergreen Edge WebView2 (`ADR-008`).
- No login/cloud sync in v1.
- Local-first SQLite; Rust owns database access (`ADR-001` canonical UTC sessions, `ADR-002` idle intervals, `ADR-006` checkpoints).
- Foreground application usage only; background processes do not accumulate app time.
- Handle idle, lock, sleep, resume, app switch, midnight, and restart recovery.
- No keystrokes, screenshots, browser URLs, document contents, or cloud activity analytics.
- Design: Soft Playful Neo-Brutalism (`ADR-004` WCAG 2.2 AA Light & Dark tokens).
- Mascot: original stylized 3D fantasy creature, rigged and animated (`ADR-007`). Do not replace it with static SVG as a shortcut.
- Figma MCP is explicitly excluded (`ADR-008`).

## Engineering rules

- Keep Windows API calls in a dedicated Rust platform module.
- Use one tracking coordinator as the source of truth for session transitions.
- Use monotonic time for elapsed duration; wall-clock for display/storage context.
- Use typed Tauri commands/events and validate all inputs.
- Use parameterized SQL and transactional writes.
- Do not put OS tracking or SQLite internals in React components.
- Do not introduce dependencies without checking need, compatibility, maintenance, and license.
- Do not use aggressive process polling when an event-based API is appropriate.
- Do not claim low RAM without release-build benchmarks.

## UI rules

- Use semantic design tokens and reusable components.
- Keep accessibility and reduced motion in scope.
- No filler badges, excessive pills, decorative technical notation, or fake statistics.
- Every data screen needs loading, empty, and error states.
- Use real data or clearly identified fixtures; never show fixtures as actual usage.

## Mascot ownership

The agent performs the complete available concept-to-GLB workflow, including automated Blender tasks where feasible. The product owner does not need to operate Blender. Report API keys, paid credits, licenses, or non-automatable steps as blockers. Never claim model/rig/animation success without inspectable files and tests.

## MCP rules

- Context7 and Playwright are preferred required workflow tools when configured.
- GitHub MCP is optional.
- Do not use or install Figma MCP.
- Do not send local user activity records or executable paths to remote MCPs.

## Completion report

For each task, report: files changed, skill names and paths actually read/applied, tests/build commands and outcomes, screenshots/benchmarks where relevant, known limitations, and TODOs. Never fabricate tests, skills, MCP use, or asset outputs.

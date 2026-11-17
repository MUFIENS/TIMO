# Timo — Product Blueprint

> **A little clarity for your digital day.**  
> A friendly, lightweight, privacy-first Digital Wellbeing desktop application for Windows.

**Document set:** Product Blueprint v1.1 (Phase 0 Refined)  
**Last updated:** 17 November 2026  
**Status:** Product direction and official branding approved; Phase 0 ADRs established for technical specification refinement before implementation.

## 1. What is Timo?

Timo helps people understand how they spend time on their Windows computer. It automatically tracks the foreground application, distinguishes active use from inactivity and system interruptions, identifies installed and standalone applications, categorizes them, and turns the resulting records into understandable daily, weekly, and monthly insights.

Timo is not employee surveillance software, a keylogger, a screenshot recorder, or a punitive productivity tool. Its purpose is awareness and healthier digital habits.

## 2. Approved product decisions

| Area | Decision |
|---|---|
| Product name | **Timo** (working brand; trademark/domain availability not yet checked) |
| Official tagline | **A little clarity for your digital day.** (**APPROVED — OWNER DECISION**) |
| Product type | Desktop Digital Wellbeing / screen-time analytics |
| Initial platform | Windows 11 x64; Windows 10 compatibility testing is best-effort only |
| Authentication | No Timo account or cloud login for v1 |
| Data model | Local-first; SQLite on the user's device |
| Tracking | Foreground application sessions; background processes are not counted as app usage |
| System states | Detect idle, lock, sleep, resume, and app switches |
| Recognition | Executable metadata, publisher, package identity, launcher manifests, curated rules, and user correction |
| Core pages | Overview, Timeline, Applications, Insights, Settings; Application Details is a nested view |
| Visual identity | **Soft Playful Neo-Brutalism** |
| Mascot | Friendly, stylized 3D fantasy creature with rigged, state-based animation |
| Desktop stack | Tauri 2, Rust, React, TypeScript, Vite, Tailwind CSS, SQLite |
| Core UI libraries | shadcn/ui + Radix UI, Motion, Recharts, Lucide React |
| 3D | Three.js + React Three Fiber + Drei; GLB assets |
| MCP | Context7 and Playwright; GitHub optional; Figma MCP is explicitly not used |
| Delivery | Windows installer, initially NSIS; signed updater only when signing material is available |

## 3. Documentation map

### Product definition
- [PRD.md](PRD.md) — concise product requirements document and product decisions.
- [PRODUCT_VISION.md](PRODUCT_VISION.md) — vision, principles, positioning, and non-goals.
- [PRODUCT_REQUIREMENTS.md](PRODUCT_REQUIREMENTS.md) — functional and non-functional requirements with IDs and acceptance criteria.
- [TARGET_AUDIENCE.md](TARGET_AUDIENCE.md) — audiences, needs, scenarios, and accessibility implications.
- [USER_FLOW.md](USER_FLOW.md) — onboarding, everyday use, app review, and settings flows.
- [APPLICATION_STRUCTURE.md](APPLICATION_STRUCTURE.md) — navigation, page responsibilities, and information architecture.

### Brand, UI, and mascot
- [DESIGN_SYSTEM.md](DESIGN_SYSTEM.md) — design tokens and reusable UI foundations.
- [DESIGN.md](DESIGN.md) — product-wide design direction and page composition principles.
- [STYLEGUIDE.md](STYLEGUIDE.md) — practical visual and frontend contribution rules.
- [MASCOT_SPECIFICATION.md](MASCOT_SPECIFICATION.md) — mascot concept, behavior, and asset acceptance criteria.
- [MASCOT_PRODUCTION_PIPELINE.md](MASCOT_PRODUCTION_PIPELINE.md) — autonomous AI-assisted 3D asset pipeline.
- [ANIMATION_SPECIFICATION.md](ANIMATION_SPECIFICATION.md) — UI motion and mascot state machine.

### Engineering
- [TECH_STACK.md](TECH_STACK.md) — detailed stack choices and rationale.
- [TECHSTACK.md](TECHSTACK.md) — quick stack reference for agents.
- [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md) — runtime architecture and component boundaries.
- [ARCHITECTURE.md](ARCHITECTURE.md) — high-level architecture map and engineering rules.
- [TRACKING_ENGINE_SPEC.md](TRACKING_ENGINE_SPEC.md) — Windows foreground, idle, lock, sleep, and session behavior.
- [APPLICATION_RECOGNITION_SPEC.md](APPLICATION_RECOGNITION_SPEC.md) — application and game identification/classification.
- [DATABASE_SCHEMA.md](DATABASE_SCHEMA.md) — SQLite entities, relations, and data lifecycle.
- [PRIVACY_SECURITY.md](PRIVACY_SECURITY.md) — privacy policy requirements and local data handling.
- [SECURITY.md](SECURITY.md) — threat model, secure implementation, and release controls.
- [PERFORMANCE_REQUIREMENTS.md](PERFORMANCE_REQUIREMENTS.md) — RAM/CPU targets and measurement plan.
- [MCP_CONFIGURATION.md](MCP_CONFIGURATION.md) — approved MCP integrations and usage boundaries.
- [adr/](adr/) — Architecture Decision Records (`ADR-001` through `ADR-008`) and [PHASE_0_DECISION_REPORT.md](PHASE_0_DECISION_REPORT.md).

### Delivery and agent operations
- [AGENTS.md](AGENTS.md) — repository-wide instructions for coding agents.
- [GEMINI.md](GEMINI.md) — guidance for Gemini-powered Antigravity agents.
- [SKILL.md](SKILL.md) — mandatory discovery, inspection, and application of installed Antigravity skills.
- [WORKFLOW.md](WORKFLOW.md) — docs-first implementation workflow and quality gates.
- [TODO.md](TODO.md) — actionable project checklist.
- [QA_TESTING_PLAN.md](QA_TESTING_PLAN.md) — verification strategy and test matrix.
- [IMPLEMENTATION_ROADMAP.md](IMPLEMENTATION_ROADMAP.md) — staged delivery plan.

## 4. Source-of-truth and conflict resolution

Use this precedence when two documents appear inconsistent:

1. Explicit user-approved decisions recorded in `PRD.md` and this README.
2. Detailed product behavior in `PRODUCT_REQUIREMENTS.md` and the relevant feature specification.
3. Architecture and implementation constraints in `SYSTEM_ARCHITECTURE.md`, `ARCHITECTURE.md`, and `TECH_STACK.md`.
4. Visual implementation rules in `DESIGN_SYSTEM.md` and `STYLEGUIDE.md`.
5. Agent execution guidance in `AGENTS.md`, `GEMINI.md`, `SKILL.md`, and `WORKFLOW.md`.

Do not silently resolve a material contradiction. Record it in `TODO.md` or an Architecture Decision Record (ADR), explain the impact, and ask the product owner only when a decision cannot safely be inferred.

## 5. Decision status vocabulary

- **Approved:** explicitly agreed by the product owner.
- **Recommended:** selected as the default proposal in this blueprint, pending implementation validation.
- **Provisional:** a testable default; benchmark or user testing may change it.
- **Future:** intentionally outside the first release.
- **Blocked:** cannot proceed until a dependency, credential, license, or product decision is available.

## 6. Important boundaries

- Do not add Figma MCP or make Figma a required dependency.
- Do not add login/cloud sync to the first release.
- Do not count background processes as application usage.
- Do not record keystrokes, screenshots, window contents, or browser URLs in v1.
- Do not classify an unknown executable as a game merely because its filename looks game-like.
- Do not claim the app is lightweight without profiling it on Windows.
- The owner does not operate Blender. The AI agent owns the mascot pipeline and must automate available tools; it must report external credentials, credits, or manual-only blockers honestly.
- Never claim an installed Antigravity skill was used unless the agent discovered, read, and applied it.

## 7. Recommended first implementation milestone

Prove a minimal native tracking spike before designing the full dashboard: detect foreground-window changes, identify process identity, detect idle/lock/sleep transitions, write accurate session intervals to a local SQLite database, and test that background applications receive no usage time. Only after this proof passes should the team build the complete UI.

## 8. References

Official technical references should be rechecked during implementation because APIs, package versions, pricing, and platform support can change:

- Tauri: https://v2.tauri.app/
- Tauri Windows installer: https://tauri.app/distribute/windows-installer/
- Microsoft `SetWinEventHook`: https://learn.microsoft.com/windows/win32/api/winuser/nf-winuser-setwineventhook
- Microsoft `GetLastInputInfo`: https://learn.microsoft.com/windows/win32/api/winuser/nf-winuser-getlastinputinfo
- Microsoft Windows 10 end of support: https://support.microsoft.com/en-us/windows/deployment/updates-lifecycle/windows-10-support-has-ended-on-october-14-2025
- Context7: https://context7.com/docs/installation
- Playwright MCP: https://github.com/microsoft/playwright-mcp
- Motion for React: https://motion.dev/docs/react
- React Three Fiber: https://r3f.docs.pmnd.rs/getting-started/introduction
- Tripo developer docs: https://developers.tripo3d.ai/en

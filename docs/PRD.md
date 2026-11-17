# PRD — Timo

**Version:** 1.1 (Phase 0 Refined)  
**Date:** 17 November 2026  
**Status:** Product direction and official tagline approved (**APPROVED — OWNER DECISION**); engineering defaults and ADRs are documented in `docs/adr/` for Phase 1–6 validation.

## 1. Product summary

Timo is a Windows desktop Digital Wellbeing application that automatically measures the time a person actively spends in foreground applications. It identifies applications, groups them into understandable categories, and presents daily, weekly, and monthly usage through an approachable interface.

**Official tagline:** **A little clarity for your digital day.** (**APPROVED — OWNER DECISION**)

## 2. Problem statement

Desktop users often have no simple, trustworthy overview of how their computer time is distributed. A list of running processes does not explain what the user actively used, and a total screen-time number does not reveal which applications dominate the day. Manual tracking is inconvenient and quickly becomes inaccurate.

## 3. Product goals

1. Make desktop usage visible with automatic foreground tracking.
2. Avoid counting background processes as active application use.
3. Detect idle, lock, sleep, resume, and application switches so durations are not inflated.
4. Recognize installed, Store, launcher-based, portable, and standalone EXE applications as accurately as possible.
5. Give users clear per-app details and a chronological usage timeline.
6. Help users reflect without shame, pressure, or punitive language.
7. Keep personal activity data local by default and keep background resource usage low.

## 4. Non-goals for v1

- macOS or Linux support.
- User accounts, cloud synchronization, social features, or leaderboards.
- Keystroke logging, screenshots, content inspection, browser URL/tab history, or employee monitoring.
- Blocking applications or enforcing hard usage limits.
- AI/LLM classification that sends executable metadata to a remote service.
- Full process inventory monitoring as a measure of usage.

## 5. Target users

Primary: students and digitally active users who use a Windows PC for mixed study, work, creative, entertainment, and gaming activities.  
Secondary: developers, designers, creators, office workers, families, and general computer users.  
The interface must remain understandable for people with different ages, levels of technical skill, and accessibility needs.

## 6. Platform and deployment

- Official initial target: Windows 11 x64.
- Windows 10: optional compatibility testing, best-effort only; Windows 10 reached end of support on 14 October 2025. Do not promise security-supported Windows 10 operation.
- No cross-platform abstraction is required in v1, but isolate OS-specific tracking code behind a Rust interface so future ports remain possible.
- Deliver as a Windows installer, initially NSIS, with optional launch-at-login and system-tray behavior.

## 7. Core user experience

Primary navigation:
1. Overview
2. Timeline
3. Applications
4. Insights
5. Settings

Application Details is a nested detail view reached from Applications or a dashboard item. Onboarding introduces Timo, explains local tracking, suggests the current Windows account name as a greeting name, allows editing/confirmation, and starts tracking with sensible defaults.

## 8. Functional scope

### Must Have
- Foreground window change detection.
- App session start/stop and accurate duration accumulation.
- Idle threshold detection (recommended default: 5 minutes; configurable).
- Lock/unlock and sleep/resume handling.
- Daily boundary handling and local-time aggregation.
- Application identity based on executable path, product/file metadata, publisher, package identity, and available platform manifests.
- Rule-based categorization with evidence and confidence levels.
- Manual correction and persistence of user overrides.
- Overview dashboard, application list, application details, timeline, weekly/monthly insights, and settings.
- Local SQLite database and data export/delete controls.
- System tray/background tracking without keeping the dashboard continuously rendered.

### Should Have
- Startup with Windows, opt-in.
- Application review center for unknown/ambiguous applications.
- User-adjustable greeting name and idle threshold.
- Light and dark themes; reduced-motion support.

### Future
- Focus sessions, goals, break reminders, app limits, advanced media-session awareness, cloud sync, and other operating systems.

## 9. Tracking definitions

- **Foreground application:** process that owns the currently active foreground window, after filtering Timo and excluded system surfaces.
- **Active application usage:** elapsed time while an eligible app is foreground, including time prior to idle threshold confirmation (`ADR-002`).
- **Idle time:** elapsed time starting exactly when the configured no-input threshold is confirmed (forward cutoff per `ADR-002`). It is displayed separately and never overlaps active app usage.
- **Locked/sleep time:** excluded from active usage and idle time. Lock/sleep transitions may appear as timeline markers but no sensitive content is recorded.
- **Daily usage:** sum of eligible active application sessions projected onto the user's local calendar day from canonical UTC sessions (`ADR-001`). Do not add idle, lock, or sleep intervals.

The default idle threshold (5 minutes, configurable in Settings) uses keyboard/mouse inactivity (`ADR-002` **Approved** forward-cutoff policy). Passive activities such as watching a video or reading can have no keyboard/mouse input and may cross the threshold if no input occurs; make this behavior transparent and easy to configure without inspecting media contents in v1.

## 10. Application recognition and categorization

Recognition precedence:
1. User override.
2. Exact curated application rule / package identity.
3. Verified product and publisher metadata combined with executable identity.
4. Platform metadata/manifests (for example Steam or Epic game manifests where available).
5. Filename/path heuristics, marked low confidence.
6. Unknown / needs review.

Default categories: Productivity, Entertainment, Gaming, Communication, System & Utilities, and Others/Uncategorized. Store application type separately from category where useful (for example, a game launcher versus an actual game). Steam/Epic launcher time must not be substituted for the foreground game's time. Only the foreground process/window is counted.

Do not assume Chrome or any multipurpose application is always productive (`ADR-003` **Approved**). Keep the 6 default categories for MVP, recognize known browsers cleanly as `app_type = browser`, default them to `Others / Uncategorized`, and persist manual user category changes.

## 11. Personalization

Timo does not require account login. During first-run onboarding, the app may read the current Windows account username locally as a suggested greeting name. Show the suggestion to the user, allow confirmation/editing, persist only the chosen display name in local preferences, and allow changing it later. If unavailable, use a neutral greeting. Never send account identity to a server.

## 12. Mascot and visual direction

- Visual style: Soft Playful Neo-Brutalism.
- Mascot: original, friendly, stylized 3D fantasy creature; compact rounded body, expressive eyes, soft matte/PBR materials, recognizable silhouette.
- AI agent owns concept, 3D generation, refinement, rigging, animation, export, optimization, and integration.
- User is not expected to open or operate Blender.
- Target asset format: optimized GLB with named animation clips and appropriate material/texture budgets.
- Never silently replace the required 3D mascot with SVG/static vector art if 3D production fails. Report blockers honestly.

## 13. Technology direction

Tauri 2 + Rust + Windows APIs + React + TypeScript + Vite + Tailwind CSS + SQLite. UI foundation: shadcn/ui and Radix primitives. Motion: Motion for React. Charts: Recharts. 3D: Three.js, React Three Fiber, Drei. Data access to SQLite remains in the Rust backend; React communicates through validated Tauri commands/events.

## 14. Privacy and security

Local-first and offline-capable. No login, cloud analytics, keystrokes, screenshots, browser URLs, document contents, or remote executable uploads in v1. Read only the metadata needed for recognition. Treat file paths and metadata as potentially sensitive. Provide user-controlled export and deletion.

## 15. Performance goals

Provisional engineering targets, not guarantees:
- Background/tray steady-state memory: target <=100 MB on the defined reference Windows 11 machine.
- Visible dashboard: target <=150 MB on the same reference machine.
- Idle CPU: near 0% averaged over a 5-minute idle window, excluding brief wake/event work.
- No continuous 3D rendering while mascot is off-screen, window hidden, minimized, or in tray.
- Foreground tracking should be event-driven; low-frequency idle sampling only as required by the idle policy.

Measure release builds on documented hardware and report median/peak results.

## 16. Success metrics

- Tracking correctness: tested interval totals match controlled test scenarios within agreed tolerance.
- No double-counting across app switches, midnight boundaries, idle, lock, and sleep.
- App recognition: precision/coverage measured against a representative labeled test corpus; unknowns remain reviewable rather than falsely classified.
- Performance: measured RAM/CPU against the provisional budget.
- Usability: first-time user can complete onboarding and find today's top apps without assistance.
- Privacy: no undisclosed network transmission of activity data.

## 17. Release acceptance

A release candidate is not accepted until the tracking engine, recognition, database recovery, privacy controls, installer/uninstaller, keyboard navigation, reduced motion, and performance test suites pass. See `QA_TESTING_PLAN.md` and `PRODUCT_REQUIREMENTS.md`.

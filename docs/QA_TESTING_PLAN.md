# QA & Testing Plan — Timo

## Test layers

### Rust unit tests
- Session state transitions.
- Monotonic duration arithmetic.
- Midnight splitting and timezone changes.
- Idle threshold behavior.
- Classification precedence and confidence.
- Stable application identity and PID reuse.
- Database repository and migration logic.

### Windows integration tests
- Foreground hook registration/unregistration.
- Foreground HWND -> PID/process identity (including UWP `ApplicationFrameHost.exe` child unwrapping and documented `CreateToolhelp32Snapshot`/`WTSEnumerateProcessesW` fallback on `ERROR_ACCESS_DENIED`).
- Idle detection (forward cutoff at threshold confirmation, `last_input_at_utc` + `confirmed_at_utc`, configurable Settings threshold, wraparound safety).
- Lock/unlock notifications.
- Suspend/resume.
- Packaged app identity where available.
- Tray lifecycle and startup behavior.

### UI tests
- Onboarding and greeting name confirmation/edit/skip.
- Overview values and empty state.
- Timeline navigation and interval selection.
- Search/filter/sort application list.
- Category correction and reset.
- Application details date range.
- Insights and settings.
- Keyboard navigation, focus, reduced motion, light/dark themes.

Use Playwright MCP for appropriate frontend/browser-based validation. Native Windows behavior must also have native integration tests; Playwright alone is insufficient.

## Critical acceptance scenarios

| ID | Scenario | Expected result |
|---|---|---|
| QA-001 | VS Code foreground for 10 min, Chrome in background | VS Code receives ~10 min; Chrome receives 0 during interval |
| QA-002 | Switch A -> B -> A | Three non-overlapping sessions with correct app attribution |
| QA-003 | Spotify playing in background while VS Code foreground | Spotify does not gain active app time |
| QA-004 | Idle threshold crossed | Pre-threshold foreground time stays active; app session stops exactly at confirmation (`T_confirm`, forward cutoff per `ADR-002`); `idle_intervals` starts at `T_confirm` with `last_input_at_utc` + `confirmed_at_utc` recorded and zero overlap |
| QA-005 | Input resumes | `idle_intervals` closes; new app session starts after foreground re-evaluation |
| QA-006 | Lock for 10 min | No application usage or idle accumulation during locked period |
| QA-007 | Sleep for 10 min | No application usage or idle accumulation during sleep |
| QA-008 | Game launched through Steam | Foreground game gets usage, Steam background does not |
| QA-009 | Standalone game EXE | Session recorded; app can be reviewed/categorized |
| QA-010 | Unknown executable / browser default | Unknown EXE tracked as `unknown` without game guessing; browser (`app_type = browser`) defaults to `Others / Uncategorized`; manual category edits persist across restarts (`ADR-003`) |
| QA-011 | Midnight crossing | Daily reports split accurately |
| QA-012 | Timo hidden to tray | Tracking continues without dashboard rendering |
| QA-013 | User quits Timo | Hooks removed and tracking stops |
| QA-014 | Database failure | Clear error and safe recovery; no fake zero data |
| QA-015 | Reduced motion | Nonessential UI/mascot animations disabled or minimized |

## Recognition corpus

Maintain a test list with app name, executable/package identity, publisher, source (installer/store/launcher/portable), expected type/category, and ground-truth notes. Include multiple versions and custom install paths. Do not store private user data in the repository.

## Accessibility and visual QA

- 100%, 125%, 150% Windows display scaling.
- Keyboard-only navigation.
- Screen reader labels for main controls.
- High contrast and focus visibility.
- Color distinction with grayscale/contrast checks.
- Reduced motion.
- Window sizes around 980×640 and smaller compact modes.

## Performance QA

Follow `PERFORMANCE_REQUIREMENTS.md`. Attach measurement report for every release candidate.

## Release gate

No P0 failures, no unresolved privacy/security issue, no false app usage during background/lock/sleep scenarios, database migration and recovery pass, installer/uninstaller pass, and mascot integration has a documented working state (or an explicitly approved deferral).

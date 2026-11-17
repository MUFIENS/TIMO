# Privacy & Data Protection — Timo

## Privacy promise

Timo tracks which application is in the foreground and how long an eligible active session lasts. It does not need to know what the user types, reads, views, or creates inside the application.

## Data collected locally

- App identity and metadata needed for recognition.
- Foreground app session start/end/duration.
- Idle, lock, unlock, suspend, resume, and tracking state transitions.
- User preferences and confirmed display name.
- Optional local diagnostics with sensitive fields redacted.

## Data explicitly not collected in v1

- Keyboard content or keystroke counts.
- Screenshots, webcam, microphone, or screen recording.
- Document contents, clipboard, or file contents.
- Browser URLs, tabs, search terms, or website content.
- Messages, contacts, or communication content.
- Cloud account identifiers or remote analytics.

## Local-first behavior

- Tracking and dashboard work offline.
- Usage database stays in the local application data directory.
- No usage-data transmission is permitted by default.
- External 3D generation is a development-time asset workflow, not runtime telemetry.
- Any future crash reporting must be opt-in, documented, and redact activity data.

## Windows username personalization

Read the current account username locally only to suggest a greeting. The first-run UI must allow the user to confirm, edit, or skip it. Persist the chosen display name locally. Do not transmit the account name. Do not assume the current Windows account holder is the current app user on a shared device.

## User controls

- Pause/resume tracking.
- Change idle threshold.
- Exclude an application.
- Correct application category.
- Change greeting name.
- Export local data.
- Delete usage history and/or app metadata (including cached PNG icons in `%LOCALAPPDATA%\Timo\icons\` per `ADR-006`).
- Exit Timo and stop tracking.

## Transparency

Explain the difference between foreground use, idle time, locked time, and sleep. State that input-based idle detection can classify passive video watching/listening as idle. Never present estimated classification as certainty.

## Privacy reviews

Before release and before adding new data collection, review:
- Data necessity and minimization.
- Filesystem access.
- Windows permissions/capabilities.
- Diagnostic logs.
- Export contents.
- Network requests and update checks.
- Data deletion completeness.

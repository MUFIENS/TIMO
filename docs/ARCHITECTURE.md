# Architecture — High-Level Rules and Map

## Architectural style

A modular monolith: one desktop product, one Rust tracking core, one local database, and a React UI. Do not introduce microservices or a local HTTP server unless a concrete requirement proves it necessary.

## Suggested repository structure

```text
/src-tauri
  /src
    /app
    /commands
    /database
      /migrations
      repository.rs
    /platform
      /windows
        foreground.rs
        idle.rs
        session.rs
        power.rs
        process.rs
    /tracking
      coordinator.rs
      state.rs
      session.rs
      clock.rs
    /recognition
      identity.rs
      metadata.rs
      platforms.rs
      classifier.rs
      rules/
    /settings
    /telemetry
/src
  /app
  /components
  /features
    /overview
    /timeline
    /applications
    /insights
    /settings
    /mascot
  /hooks
  /lib
  /styles
  /types
/docs (or project root documentation)
```

This is a suggested structure; align with the actual Tauri template and avoid creating empty folders without implementation need.

## Dependency direction

- `platform::windows` may emit normalized events; it must not depend on React or page modules.
- `tracking` consumes normalized events and owns duration/session rules.
- `recognition` identifies apps; it does not decide elapsed time.
- `database` persists canonical records; it does not own product policy.
- `commands` exposes validated operations to Tauri.
- `features/*` consume typed UI services and never call Windows APIs directly.

## State ownership

- Tracking/session state: Rust only.
- Persistent app/category/preferences state: SQLite via Rust.
- Ephemeral UI state: React/Zustand.
- Derived analytics: query/service layer; do not persist redundant totals unless justified by profiling and reconciliation tests.

## Error model

Use typed, user-safe error codes at the UI boundary. Keep detailed diagnostics in local logs with sensitive data redacted. Errors must not be silently converted to empty data.

## Architecture decision records

Phase 0 foundational ADRs live in [`docs/adr/`](./adr/) (`ADR-001` through `ADR-008`). Create an additional ADR in `docs/adr/` when changing:
- Tauri vs another desktop shell.
- Tracking model, midnight projection (`ADR-001`), idle cutoff (`ADR-002`), or Windows API strategy.
- Database library, schema ownership, or identity/recovery rules (`ADR-006`).
- Category defaults (`ADR-003`), theme/contrast tokens (`ADR-004`), or tray memory lifecycle (`ADR-005`).
- Remote/cloud data handling.
- Mascot format/renderer/toolchain (`ADR-007`).
- Any dependency or MCP configuration (`ADR-008`) that affects licensing, performance, or long-term maintenance.

# Technology Stack — Detailed

## Desktop shell

**Tauri 2** is the desktop framework. It provides the native application shell and a controlled bridge between the React UI and Rust commands/events. Tauri is not, by itself, proof of low memory usage; WebView2 and 3D rendering must be measured.

## Backend and Windows integration

**Rust** owns tracking, process/window inspection, classification, SQLite, preferences, and native lifecycle integration.

Candidate crates (verify current versions and maintenance before adding):
- `windows` for Windows API bindings.
- `rusqlite` for embedded SQLite access and transactions.
- `serde` / `serde_json` for typed command payloads.
- `thiserror` or `anyhow` for error handling (use typed errors at application boundaries).
- `tracing` and a rotating file appender for diagnostics; redact sensitive paths.
- `crossbeam-channel` or Tokio channels for event passing where needed; avoid unnecessary runtimes.

Use a dedicated Windows event-hook thread with the message pump required by the selected hook implementation. Keep OS-specific code behind a small `platform::windows` boundary.

## Frontend

- React + TypeScript.
- Vite for frontend build/development.
- Tailwind CSS for styling.
- shadcn/ui built on Radix primitives for accessible controls.
- Motion for React for purposeful UI animation.
- Recharts for standard analytics charts.
- Three.js + React Three Fiber + Drei for the 3D mascot.
- Zustand for small, client-side UI state where appropriate.
- TanStack Table and/or TanStack Virtual for large application lists.
- date-fns for date formatting/calculation; centralize timezone-aware business rules in tested utilities.
- Lucide React for consistent icons.
- Zod for validating boundary data where useful.

## Database

SQLite, accessed only by Rust backend. Recommended configuration: foreign keys enabled, WAL mode after concurrency tests, bounded busy timeout, schema migrations, transactional writes, and periodic integrity/backup strategy.

Do not let arbitrary React components open the database directly. Expose typed Tauri commands for queries and mutations.

## Tauri plugins/capabilities

Evaluate only required plugins: autostart, updater, and any required dialog/fs integration. Keep capabilities least-privilege. Do not add a SQL plugin if all DB access is intentionally owned by Rust.

## Packaging

NSIS installer initially. MSI may be added after testing. Signed updates require a release signing key and secure private-key handling; updater is not enabled until signing and distribution are configured.

## Version policy

Do not copy version numbers from old prompts or examples. At project initialization, use Context7 and official package registries/docs to select mutually compatible stable versions, record them in lockfiles, and avoid floating major versions. Record the verified versions in `package.json`, `Cargo.toml`, lockfiles, and a build metadata report.

## Explicit exclusions

- No Electron for v1 unless a documented technical constraint invalidates Tauri.
- No remote database/cloud account for v1.
- No Figma MCP.
- No excessive UI component/animation libraries.

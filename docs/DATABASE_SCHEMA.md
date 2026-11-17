# Database Schema — SQLite

## Principles

- SQLite is local and owned by the Rust backend.
- Canonical records are usage sessions, application identities, categories, preferences, and classification evidence.
- Compute daily/weekly summaries from canonical records initially; add cached aggregates only if profiling justifies them and reconciliation is tested.
- Use schema migrations from the first commit.
- Enable foreign keys; evaluate WAL and busy timeout in the real workload.

## Core tables

### `applications`

| Column | Type | Notes |
|---|---|---|
| `id` | TEXT primary key | Stable UUID/app key |
| `stable_app_key` | TEXT unique not null | Deterministic identity key (`pkg:...`, `meta:...`, `exe:...` per `ADR-006`) |
| `display_name` | TEXT | User-facing app name |
| `executable_name` | TEXT nullable | Example `Code.exe` |
| `executable_path` | TEXT nullable | Local only; treat as sensitive; redacted in default exports |
| `icon_cache_path` | TEXT nullable | Local relative icon cache token/path (`ADR-006`); deleted on purge |
| `package_family_name` | TEXT nullable | Store/package identity |
| `aumid` | TEXT nullable | Application user model ID when available |
| `publisher` | TEXT nullable | Metadata evidence |
| `product_name` | TEXT nullable | PE version resource |
| `app_type` | TEXT nullable | game, game launcher, browser, editor/IDE, media player, communication client, utility |
| `category_id` | TEXT nullable FK | Current category |
| `classification_state` | TEXT | `confirmed_user`/`high`/`medium`/`low`/`unknown` (curated neutral browsers use `low` category certainty for Review Queue visibility per `ADR-003`) |
| `source_rule_id` | TEXT nullable | Rule/evidence reference |
| `first_seen_at` | TEXT | UTC ISO-8601 |
| `last_seen_at` | TEXT | UTC ISO-8601 |
| `is_excluded` | INTEGER | Boolean |

### `categories`

`id`, `name`, `sort_order`, `color_token`, `icon_name`, `is_system`, `created_at`. Seed the initial 6 categories (`Productivity`, `Entertainment`, `Gaming`, `Communication`, `System & Utilities`, `Others / Uncategorized`); avoid storing arbitrary color values from untrusted UI input.

### `usage_sessions` (`ADR-001`, `ADR-002`, `ADR-006`)

Canonical, unsplit active foreground sessions stored in UTC with monotonic durations:

| Column | Type | Notes |
|---|---|---|
| `id` | TEXT primary key | UUID |
| `application_id` | TEXT FK | Application record |
| `started_at_utc` | TEXT | UTC ISO-8601 timestamp |
| `ended_at_utc` | TEXT nullable | Null only while actively tracking |
| `duration_ms` | INTEGER | Canonical monotonic elapsed duration on close (`>= 0`) |
| `process_id` | INTEGER nullable | Diagnostic only; not stable identity |
| `process_started_at_utc` | TEXT nullable | Disambiguates PID reuse (`TR-002`) |
| `timezone_id` | TEXT | Capture-time Windows/IANA timezone identifier |
| `utc_offset_minutes` | INTEGER | Capture-time local UTC offset in minutes |
| `last_checkpoint_utc` | TEXT | Heartbeat timestamp updated every 30s & on transitions (`ADR-006`) |
| `checkpoint_monotonic_ms` | INTEGER | Last reliable monotonic elapsed ms for crash recovery (`>= 0`) |
| `recovery_closed` | INTEGER | Boolean (`1` if closed by startup recovery after crash) |
| `created_at` | TEXT | UTC |

Per `ADR-001`, daily and midnight-crossing segments are **derived deterministically at query time** against local calendar UTC boundaries rather than split at write time. Do not store write-time `local_date` segments on `usage_sessions`.

### `idle_intervals` (`ADR-002`, `ADR-006`)

Dedicated intervals for confirmed idle periods, kept strictly distinct from active application usage:

| Column | Type | Notes |
|---|---|---|
| `id` | TEXT primary key | UUID |
| `started_at_utc` | TEXT | Effective idle start (`max(last_input_time, prior_session_start)`) |
| `confirmed_at_utc` | TEXT | UTC timestamp when threshold was crossed |
| `ended_at_utc` | TEXT nullable | Null while currently `IDLE` |
| `duration_ms` | INTEGER | Monotonic idle duration on close (`>= 0`) |
| `end_reason` | TEXT nullable | `user_input`, `locked`, `suspended`, `tracking_paused`, `app_exit`, `crash_recovery` |
| `timezone_id` | TEXT | Capture-time timezone identifier |
| `last_checkpoint_utc` | TEXT | Heartbeat timestamp updated every 30s while open |
| `checkpoint_monotonic_ms` | INTEGER | Monotonic elapsed ms for crash recovery |
| `recovery_closed` | INTEGER | Boolean |
| `created_at` | TEXT | UTC |

### `activity_events`

`id`, `event_type` (`locked`, `unlocked`, `suspended`, `resumed`, `tracking_paused`, `tracking_resumed`, `clock_anomaly`), `occurred_at_utc`, `timezone_id`, `metadata_json` (strict allowlist; no window titles/content), `created_at`. Note: Idle spans live in `idle_intervals` (`ADR-002`) while discrete system transitions remain in `activity_events`.

### `classification_evidence`

`id`, `application_id`, `evidence_type`, `evidence_value_redacted`, `rule_id`, `confidence_state`, `observed_at_utc`. Avoid storing full sensitive paths redundantly.

### `user_preferences`

Key/value or typed columns for `display_name`, `idle_threshold_seconds`, `launch_at_login`, `start_minimized`, `theme`, `reduced_motion`, `tracking_enabled`, and schema-level preference version. Do not store Windows account credentials or tokens.

### `schema_migrations`

Migration ID, applied timestamp, checksum/version.

## Relationships

- One category has many applications.
- One application has many usage sessions and classification evidence records.
- User preferences are device-local and do not require a user/account table.

## Indexes

- `usage_sessions(application_id, started_at_utc)`
- `usage_sessions(started_at_utc, ended_at_utc)`
- `idle_intervals(started_at_utc, ended_at_utc)`
- `applications(category_id, display_name)`
- `activity_events(occurred_at_utc)`
- Unique `applications(stable_app_key)` (`ADR-006`)

## Data integrity

- `duration_ms >= 0` and `checkpoint_monotonic_ms >= 0`.
- End time must not precede start time (`ended_at_utc >= started_at_utc`).
- Prevent overlapping active sessions or overlapping active idle intervals through coordinator logic and tests; a database constraint/transaction guard may provide defense in depth.
- Use transactions for session close + next session start (or session close + `idle_intervals` start).
- Handle interrupted open sessions and open idle intervals on startup using `last_checkpoint_utc` / `checkpoint_monotonic_ms` (`ADR-006`).

## Retention and export

No automatic deletion by default. Provide user-initiated delete-all and export. Export should include a schema version and timezone definitions; redact full executable paths by default unless user explicitly chooses full-detail export.

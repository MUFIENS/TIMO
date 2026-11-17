# ADR-001: Midnight Session Handling & Canonical Time Representation

**Date:** 2026-11-17  
**Status:** Recommended  
**Owners:** Windows Systems Engineering, Database Architecture  

---

## 1. Context

Timo tracks continuous foreground application usage sessions on Windows (`PRODUCT_REQUIREMENTS.md` `TR-003`, `TR-009`) and displays daily totals, chronological daily timelines, and weekly/monthly insights (`UI-001`, `UI-006`, `UI-007`). Earlier specifications contained conflicting guidance: `SYSTEM_ARCHITECTURE.md` implied write-time day-boundary splitting, `DATABASE_SCHEMA.md` included a `local_date` column on `usage_sessions` while warning not to mix write-time and query-time splitting, and `TRACKING_ENGINE_SPEC.md` left the choice between write-time splitting and query-time splitting open.

## 2. Problem

Splitting active sessions at write time introduces timer race conditions around `00:00:00` local time, creates artificial session breaks across midnight, complicates DST ("spring forward" 23-hour days and "fall back" 25-hour days with ambiguous duplicate `01:00`–`02:00` local hours), and embeds mutable local-calendar assumptions directly into historical rows if the user changes Windows timezones while traveling.

## 3. Options Considered

1. **Option A: Write-time midnight splitting** (close the open session at `23:59:59.999` local and insert a second segment at `00:00:00.000` local).
2. **Option B: Canonical unsplit UTC sessions + deterministic query-time day segmentation** (store one continuous session with UTC timestamps and monotonic `duration_ms`; slice against local calendar boundaries when querying Overview, Timeline, or Insights).

## 4. Technical Trade-offs

| Criterion | Option A: Write-time Split | Option B: Canonical UTC + Query-time Split |
|---|---|---|
| Midnight timer race & sleep edge cases | High complexity (must synthesize missed midnight splits after sleep/crash) | Zero midnight write race; suspend/recovery logic stays clean |
| DST spring-forward / fall-back | Ambiguous local `01:30` during fall-back can mis-order segments | Exact UTC intervals never collide; local day boundaries map cleanly to UTC instants |
| Timezone travel | Historical `local_date` columns freeze stale offsets | Queries project UTC spans cleanly into current or specified local calendar rules |
| SQL range query simplicity | Simple `WHERE local_date = ?` | Requires overlap predicate `started_at_utc < day_end_utc AND (ended_at_utc > day_start_utc OR ended_at_utc IS NULL)` indexed on `(started_at_utc, ended_at_utc)` |
| Timeline continuity | A 23:45–00:20 session becomes two separate DB rows | One canonical source-of-truth session; UI displays cleanly per day |

## 5. Recommended Decision

Adopt **Option B: Canonical unsplit UTC sessions with monotonic `duration_ms` and deterministic query-time local day segmentation**.

- Persist each continuous foreground interval as **one** `usage_sessions` row containing:
  - `started_at_utc` (`TEXT`, UTC ISO-8601)
  - `ended_at_utc` (`TEXT` nullable only while active)
  - `duration_ms` (`INTEGER`, strictly derived from monotonic clock `Instant` / `QueryPerformanceCounter`, never `ended_at_utc - started_at_utc`)
  - `timezone_id` (`TEXT`, capture-time Windows/IANA timezone identifier for audit/export provenance)
  - `utc_offset_minutes` (`INTEGER`, capture-time local UTC offset for historical export transparency)
- Remove write-time `local_date` segmentation from `usage_sessions` so write-time and query-time models are never mixed (`DATABASE_SCHEMA.md`).
- Derive daily segments in a centralized, tested Rust reporting/query module (`tracking::calendar` or `database::queries`) that converts a target local calendar date `[YYYY-MM-DD 00:00:00, next_day 00:00:00)` under the active local timezone rule into exact UTC half-open bounds `[day_start_utc, day_end_utc)` and slices intersecting sessions (`max(session_start, day_start)` to `min(session_end_or_now, day_end)`).
- Keep total-reconciliation invariant: for any closed session where wall-clock span equals monotonic `duration_ms`, the sum of sliced daily millisecond totals equals `duration_ms`. When a wall-clock anomaly occurs (e.g., manual NTP clock jump mid-session), scale or clamp sliced portions proportionally against canonical `duration_ms` so daily totals never exceed actual monotonic elapsed time.

## 6. Decision Status

**Recommended** (engineering specification resolved; ready for Phase 1 implementation validation and unit tests).

## 7. Consequences

- **Positive:** Eliminates synthetic midnight write timers, handles 23h/25h DST transitions accurately in UTC, simplifies crash recovery, and ensures a single canonical truth in SQLite.
- **Negative / Risk:** All SQL day-level queries must use half-open UTC range predicates (`started_at_utc < ? AND (ended_at_utc > ? OR ended_at_utc IS NULL)`) and Rust timezone conversion logic rather than naive `GROUP BY local_date`. Long multi-day open sessions are prevented separately by `ADR-002` (idle), lock, and sleep handlers.

## 8. Implementation Implications

- Update `DATABASE_SCHEMA.md` to remove `local_date` from `usage_sessions` and ensure index `idx_usage_sessions_range(started_at_utc, ended_at_utc)` supports overlap queries.
- Centralize all local-day-to-UTC range resolution inside Rust using a maintained timezone crate (e.g., `chrono-tz` / `jiff` / `iana-time-zone`, verified via official docs at Phase 0/1).
- Never compute daily totals in React; React receives already-sliced daily/timeline segments in typed IPC payloads.

## 9. Validation Requirements

Rust unit tests must prove:
1. Session crossing `23:50` to `00:15` local splits accurately across Day N (`10m`) and Day N+1 (`15m`) with zero overlap and zero lost milliseconds.
2. Spring-forward DST day (23-hour local day) and fall-back DST day (25-hour local day) sum accurately without negative or duplicate intervals.
3. Manual system wall-clock jump mid-session preserves canonical monotonic `duration_ms` and clamps slice totals cleanly.
4. `Overview` daily total equals `Timeline` active segment sum equals `Applications` day sum.

## 10. Related Documents

- [`TRACKING_ENGINE_SPEC.md`](../TRACKING_ENGINE_SPEC.md)
- [`DATABASE_SCHEMA.md`](../DATABASE_SCHEMA.md)
- [`SYSTEM_ARCHITECTURE.md`](../SYSTEM_ARCHITECTURE.md)
- [`PRODUCT_REQUIREMENTS.md`](../PRODUCT_REQUIREMENTS.md) (`TR-009`, `UI-001`, `UI-006`)
- [`QA_TESTING_PLAN.md`](../QA_TESTING_PLAN.md) (`QA-011`)

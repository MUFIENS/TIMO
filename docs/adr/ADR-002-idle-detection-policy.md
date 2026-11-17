# ADR-002: Idle Detection Policy & Interval Persistence

**Date:** 2026-11-17  
**Status:** Approved (Owner Decision on forward cutoff policy and configurable 5-minute default; numeric 5-minute default remains user-configurable in Settings)  
**Owners:** Product Owner, Windows Systems Engineering  

---

## 1. Context

Timo tracks foreground application usage using Windows `EVENT_SYSTEM_FOREGROUND` (`TR-001`) and `GetLastInputInfo` (`TR-005`) with a configurable default idle threshold of 5 minutes (`300` seconds). Earlier drafts needed an explicit owner policy on whether crossing the idle threshold retroactively closes the active session at `last_input_time` or closes it forward at `threshold_crossed_at` (`confirmed_at_utc`), plus a persistence model that separates active usage from idle intervals across midnight, lock, sleep, and crash recovery.

## 2. Problem

Input-based idle detection must balance avoiding break inflation against avoiding unfair loss of legitimate passive computer use (reading articles, thinking, reviewing documentation, or watching videos without mouse/keyboard input):
- Retroactive cutoff deletes up to 5 minutes of foreground time immediately when the threshold triggers, creating a sudden "cliff" during passive reading or watching.
- Forward cutoff keeps the pre-threshold window in active foreground usage while cleanly switching to an idle interval at confirmation time, requiring strict zero-overlap guarantees and clear provenance (`last_input_at_utc` + `confirmed_at_utc`).
- Storing only point `activity_events` (`idle_started`, `idle_ended`) makes idle duration aggregation, lock/sleep clipping, and crash recovery fragile.

## 3. Options Considered

1. **Option A: Retroactive cutoff at `last_input_time` once threshold is confirmed.**
2. **Option B: Forward cutoff at `threshold_crossed_at` (`confirmed_at_utc`) while recording both `last_input_at_utc` and `confirmed_at_utc`.**

## 4. Technical Trade-offs

| Aspect | Option A: Retroactive Cutoff (`last_input_time`) | Option B: Forward Cutoff (`confirmed_at_utc`) |
|---|---|---|
| Passive reading / video / thinking | Crossing 5m abruptly erases the prior 5m of legitimate attention ("cliff effect") | Pre-threshold time remains counted as foreground active usage; predictable to users |
| Break accounting | Excludes the 5m pre-idle window once threshold is crossed | Counts up to `idle_threshold` before confirmation as foreground, then isolates all subsequent inactivity as idle |
| Timeline & UI live consistency | Live "Today's active time" must jump backward by 5m when idle triggers | Monotonically consistent: active time never jumps backward on idle transition |
| Focus-without-input edge cases | Requires clamping `max(last_input_time, session_start)` to prevent negative durations | Natural monotonic progression (`session.ended_at_utc == idle.started_at_utc == confirmed_at_utc`) |
| Audit transparency | Only one boundary recorded unless metadata added | Persists both `last_input_at_utc` and `confirmed_at_utc` for clarity and diagnostics |

## 5. Recommended Decision

Adopt **Option B (Approved Owner Decision): Forward Cutoff at Idle Threshold Confirmation**:

1. **Default & Configurability:**
   - Default `idle_threshold_seconds = 300` (5 minutes), configurable by the user in Settings (`TR-010`).
   - Sample `GetLastInputInfo` at a bounded low frequency (~1 Hz) using 32-bit wraparound-safe math: `current_tick32.wrapping_sub(last_input_tick32)` against `GetTickCount()`, protecting against the 49.7-day `DWORD` rollover (`TRACKING_ENGINE_SPEC.md` §3).
2. **Forward Cutoff & Transition Boundary:**
   - Suppose last keyboard/mouse input occurred at `T_input` (`last_input_at_utc`) and inactivity equals or exceeds the configured threshold at `T_confirm` (`confirmed_at_utc`).
   - **Time prior to `T_confirm` remains counted as foreground active usage.**
   - **Idle begins at `T_confirm`:**
     - Close the open `usage_sessions` row at `ended_at_utc = T_confirm`, recording monotonic `duration_ms` up to `T_confirm`.
     - Open an `idle_intervals` row at `started_at_utc = T_confirm` with `confirmed_at_utc = T_confirm`, `last_input_at_utc = T_input`, and `ended_at_utc = NULL`.
   - **Zero Double-Counting Invariant:** `usage_sessions.ended_at_utc <= idle_intervals.started_at_utc` (exact boundary handoff at `T_confirm`), guaranteeing zero overlapping duration between active usage and idle intervals.
3. **Idle Interval Exit Transitions:**
   - **Fresh user input (`IDLE -> TRACKING_APP`):** Close `idle_intervals` at `T_resume` (`end_reason = 'user_input'`), compute monotonic `duration_ms`, re-query foreground HWND, and open a new eligible app session starting at `T_resume`.
   - **Lock (`IDLE -> LOCKED`) or Suspend (`IDLE -> SUSPENDED`):** Immediately close `idle_intervals` at `T_lock` / `T_suspend` (`end_reason = 'locked'` or `'suspended'`) so locked/sleep hours are never counted as idle time.
   - **Pause / Quit / Crash Recovery:** Close `idle_intervals` at pause/quit time or `last_checkpoint_utc` (`end_reason = 'tracking_paused' | 'app_exit' | 'crash_recovery'`).
4. **Midnight Splitting & Active vs Idle Reconciliation:**
   - Apply `ADR-001` canonical UTC query-time day slicing to `idle_intervals` alongside `usage_sessions` so daily `active_ms + idle_ms` never overlap and never duplicate across midnight or DST changes.
5. **Transparency:**
   - Onboarding and Settings explain clearly that Timo uses keyboard/mouse inactivity with a configurable threshold (`5 minutes` default) and records idle periods separately from active usage (`PRD.md` §9, `PRIVACY_SECURITY.md`).

## 6. Decision Status

**Approved** (Owner confirmed forward cutoff policy, 5-minute default configurable in Settings, persistence of both `last_input_at_utc` and `confirmed_at_utc`, and strict non-overlapping active/idle accounting).

## 7. Consequences

- **Positive:** Protects passive reading, thinking, and short media viewing from sudden retroactive time loss; eliminates backward jumps in live Overview counters; keeps exact audit metadata (`last_input_at_utc` and `confirmed_at_utc`); isolates locked/sleep time cleanly via dedicated `idle_intervals`.
- **Negative / Risk:** Every walk-away break counts the threshold window (e.g., up to 5 minutes) as active time in the foreground window before idle starts (mitigated by allowing users to lower the threshold in Settings, e.g., to 1–3 minutes if desired).

## 8. Implementation Implications

- Implement `idle_intervals` table (`id`, `started_at_utc`, `confirmed_at_utc`, `last_input_at_utc`, `ended_at_utc`, `duration_ms`, `end_reason`, `timezone_id`, `last_checkpoint_utc`, `checkpoint_monotonic_ms`, `recovery_closed`, `created_at`) in `DATABASE_SCHEMA.md`.
- Keep `activity_events` for discrete markers (`locked`, `unlocked`, `suspended`, `resumed`, `tracking_paused`, `tracking_resumed`, `clock_anomaly`).
- Enforce single-transaction handoff in `TrackingCoordinator` when transitioning `TRACKING_APP -> IDLE` at `T_confirm`.

## 9. Validation Requirements

Rust unit and Windows integration tests must prove:
1. Active app from `10:00` with last input at `10:02` and 5-min threshold (`300s`): idle confirms at `10:07`; `usage_sessions` closes at `10:07` (`duration = 7m`), `idle_intervals` starts at `10:07` with `last_input_at_utc = 10:02` and `confirmed_at_utc = 10:07`, with `0 ms` overlap.
2. Changing idle threshold in Settings persists locally and governs subsequent inactivity transitions.
3. Simulated 49.7-day `GetTickCount()` wraparound (`0xFFFF_F000` -> `0x0000_2000`) computes positive elapsed idle milliseconds accurately.
4. Lock or Sleep occurring while `IDLE` closes `idle_intervals` immediately without adding locked/sleep duration to idle totals.
5. `active_ms` and `idle_ms` intervals never overlap across any sequence (`QA-004`, `QA-005`, `QA-006`, `QA-007`).

## 10. Related Documents

- [`TRACKING_ENGINE_SPEC.md`](../TRACKING_ENGINE_SPEC.md)
- [`DATABASE_SCHEMA.md`](../DATABASE_SCHEMA.md)
- [`PRODUCT_REQUIREMENTS.md`](../PRODUCT_REQUIREMENTS.md) (`TR-005`, `TR-010`)
- [`PRD.md`](../PRD.md) (§9)
- [`PRIVACY_SECURITY.md`](../PRIVACY_SECURITY.md)
- [`QA_TESTING_PLAN.md`](../QA_TESTING_PLAN.md) (`QA-004`, `QA-005`, `QA-006`, `QA-007`)

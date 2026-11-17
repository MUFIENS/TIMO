# ADR-002: Idle Detection Policy & Interval Persistence

**Date:** 2026-11-17  
**Status:** Provisional (Requires Owner Confirmation on retroactive vs forward cutoff trade-off)  
**Owners:** Product Owner, Windows Systems Engineering  

---

## 1. Context

Timo tracks foreground application usage using Windows `EVENT_SYSTEM_FOREGROUND` (`TR-001`) and `GetLastInputInfo` (`TR-005`) with a configurable default idle threshold of 5 minutes (`300` seconds). Earlier documents left two aspects ambiguous: (1) whether crossing the idle threshold retroactively ends the active application session at `last_input_time` or ends it at `threshold_crossed_at`, and (2) how idle intervals are stored and reconciled across midnight, lock, sleep, and crash recovery.

## 2. Problem

Input-based idle detection faces an unavoidable trade-off between **over-counting breaks** and **under-counting passive consumption**:
- If an app session closes only when the 5-minute threshold triggers (`10:05`), every break adds 5 phantom active minutes to whatever window happened to be open.
- If an app session closes retroactively at `last_input_time` (`10:00`), genuine breaks are cleanly separated from active time, but passive reading or watching a 6-minute video without moving the mouse loses those 5 minutes abruptly once the threshold crosses (a "cliff" between 4m59s and 5m01s).
- Furthermore, storing only point `activity_events` (`idle_started`, `idle_ended`) makes idle duration aggregation, lock/sleep clipping, and crash recovery brittle.

## 3. Options Considered

1. **Option A: Retroactive cutoff at `last_input_time` once threshold is confirmed** (capped safely at `current_session.started_at_utc` and stored as canonical idle interval).
2. **Option B: Forward cutoff at `threshold_crossed_at`** (keep the 5-minute wait inside active app time; start idle at `T + threshold`).

## 4. Technical Trade-offs

| Aspect | Option A: Retroactive Cutoff (`last_input_time`) | Option B: Forward Cutoff (`threshold_crossed_at`) |
|---|---|---|
| Break inflation | Zero inflation: walking away for 15m logs 15m idle, 0m phantom app use | Inflates active app totals by up to `idle_threshold` on every single break |
| Passive reading / video cliff | Crossing 5m removes the trailing 5m from active total; recoverable only by raising threshold | Keeps trailing 5m, but consistently overstates genuine AFK breaks |
| Focus-without-input edge case | If a popup steals focus at `10:02` while `last_input_time` was `09:58`, raw backdating precedes `started_at` unless clamped to `max(last_input_time, session.started_at)` | No backdate clamp needed |
| User mental model | Matches "Idle since 10:00 (after 5m inactivity)" | Confusing timeline showing 5m of "active" typing when user left at 10:00 |

## 5. Recommended Decision

Adopt **Option A (Provisional): Retroactive session closure at `last_input_time` once the configured idle threshold is confirmed**, paired with explicit persistence rules:

1. **Threshold & Polling:**
   - Default `idle_threshold_seconds = 300` (5 minutes), configurable by user (`TR-010`).
   - Sample `GetLastInputInfo` at ~1 Hz (`~1000 ms` bounded timer) using 32-bit wraparound-safe math: `(current_tick32.wrapping_sub(last_input_tick32))` against `GetTickCount()`, protecting against the 49.7-day `DWORD` rollover (`TRACKING_ENGINE_SPEC.md` §3).
2. **Retroactive Cutoff & Clamp Rule:**
   - Suppose last input occurred at `T_input` and idle is confirmed at `T_confirm` (`T_confirm - T_input >= threshold`).
   - Effective cutoff time `T_cut = max(T_input, active_session.started_at_monotonic)`.
   - Close `usage_sessions` with `ended_at_utc` corresponding to `T_cut` and `duration_ms = T_cut - active_session.started_at_monotonic`.
   - Note: If `T_cut == active_session.started_at_monotonic` (e.g., a background popup switched focus during an already-inactive period and produced `0 ms` active time), discard or avoid opening a zero-duration ghost session.
   - Open an **`idle_intervals`** record with `started_at_utc` at `T_cut`, `confirmed_at_utc` at `T_confirm`, and `ended_at_utc = NULL`.
3. **Idle Interval Exit Transitions:**
   - **Fresh user input (`IDLE -> TRACKING_APP`):** Close `idle_intervals` at `T_resume` (`end_reason = 'user_input'`), compute monotonic `duration_ms`, re-query foreground HWND, and open new eligible app session.
   - **Lock (`IDLE -> LOCKED`) or Suspend (`IDLE -> SUSPENDED`):** Immediately close `idle_intervals` at `T_lock` / `T_suspend` (`end_reason = 'locked'` or `'suspended'`) so locked/sleep hours are never counted as idle time.
   - **Pause / Quit / Crash Recovery:** Close `idle_intervals` at pause/quit time or `last_checkpoint_utc` (`end_reason = 'tracking_paused' | 'app_exit' | 'crash_recovery'`).
4. **Midnight Splitting & Active vs Idle Reconciliation:**
   - Apply the same query-time canonical UTC slicing rule from `ADR-001` to `idle_intervals` so daily `active_ms + idle_ms` never overlap and never exceed 24 hours (accounting for DST).
5. **Transparency Mandate:**
   - UI onboarding and Settings idle-threshold helper text must plainly disclose that Timo detects keyboard/mouse input inactivity, so passive video watching or reading without input may register as idle (`PRD.md` §9, `PRIVACY_SECURITY.md`).

## 6. Decision Status

**Provisional** — **Requires Owner Confirmation** (Confirm whether retroactive trimming to `last_input_time` is preferred over forward cutoff at `threshold_crossed_at` given the passive-media/reading cliff trade-off).

## 7. Consequences

- **Positive:** Prevents systematic +5-minute inflation on every walk-away break; keeps Timeline faithful to when the user actually stopped interacting; handles lock/sleep/crash cleanly via dedicated `idle_intervals` table.
- **Negative / Risk:** Slight UI complexity if live "Today's active time" ticks forward during the 5-minute pre-idle grace period and then adjusts back by 5 minutes once idle confirms (mitigated by refreshing overview summary cleanly when state transitions to `IDLE`).

## 8. Implementation Implications

- Add `idle_intervals` table (`id`, `started_at_utc`, `confirmed_at_utc`, `ended_at_utc`, `duration_ms`, `end_reason`, `timezone_id`, `created_at`) to `DATABASE_SCHEMA.md`.
- Keep `activity_events` for discrete markers (`locked`, `unlocked`, `suspended`, `resumed`, `tracking_paused`, `tracking_resumed`).
- Clamp `T_cut` strictly to `>= active_session.started_at` so `duration_ms >= 0` always holds.

## 9. Validation Requirements

Unit and native integration tests must cover:
1. Active app for 3m, no input for 5m (`T_input = 10:00`, `T_confirm = 10:05`) -> active session records `3m` ending at `10:00`; idle interval begins at `10:00`.
2. Simulated 49.7-day `GetTickCount()` wraparound (`0xFFFF_F000` -> `0x0000_2000`) computes exact positive elapsed idle milliseconds.
3. Lock or Sleep occurring while already `IDLE` closes `idle_intervals` immediately without counting locked/asleep duration as idle.
4. App focus switch without input (`T_input < session_start`) clamps safely without negative `duration_ms`.

## 10. Related Documents

- [`TRACKING_ENGINE_SPEC.md`](../TRACKING_ENGINE_SPEC.md)
- [`DATABASE_SCHEMA.md`](../DATABASE_SCHEMA.md)
- [`PRD.md`](../PRD.md) (§9)
- [`PRIVACY_SECURITY.md`](../PRIVACY_SECURITY.md)
- [`QA_TESTING_PLAN.md`](../QA_TESTING_PLAN.md) (`QA-004`, `QA-005`, `QA-006`, `QA-007`)

# Tracking Engine Specification — Windows

## 1. Objective

Measure foreground application usage accurately while excluding background processes, idle time, locked sessions, and sleep time.

## 2. Definitions

- **Foreground window:** HWND returned as the active foreground window after validation.
- **Process instance:** process ID plus process creation identity and resolved executable/package evidence; PID alone is insufficient because it can be reused.
- **Application:** stable Timo record that may represent multiple process instances/windows.
- **App session:** continuous eligible foreground period attributed to one application.
- **Idle:** no qualifying user input for at least the configured threshold.
- **Locked:** Windows session is locked; no app usage is counted.
- **Suspended:** PC is in a sleep/suspend transition; no app usage is counted.

## 3. Windows event strategy

### Foreground changes

Use `SetWinEventHook` for `EVENT_SYSTEM_FOREGROUND` with an appropriate out-of-context hook and a dedicated thread/message loop. Resolve the HWND to PID and gather process identity. Verify API flags, callback lifetime, thread requirements, and cleanup against current Microsoft documentation.

### Idle

Use documented `GetLastInputInfo` to obtain last input tick. Check at a low-frequency interval (initial recommendation: approximately once per second; tune by measurement). Convert tick differences using 32-bit wraparound-safe math (`current_tick32.wrapping_sub(last_input_tick32)`) against `GetTickCount()` to account for the 49.7-day rollover (`ADR-002`). Idle threshold defaults to 5 minutes (`300` seconds) and is user-configurable via Settings (`TR-010`).

Per `ADR-002` (**Approved** forward-cutoff policy):
- Time prior to idle confirmation remains counted as foreground active usage.
- Idle starts **exactly when the threshold is confirmed (`T_confirm`)**, never retroactively at `last_input_time`.
- On confirmation at `T_confirm`, close the open `usage_sessions` row at `ended_at_utc = T_confirm` (with monotonic `duration_ms` up to `T_confirm`) and open an `idle_intervals` row with `started_at_utc = T_confirm`, `confirmed_at_utc = T_confirm`, and `last_input_at_utc = T_input`, guaranteeing zero overlapping duration between active usage and idle intervals.
- Lock or sleep while `IDLE` immediately closes the open idle interval so locked/sleep duration is never counted as idle.

### Lock and unlock

Use session notifications such as `WTSRegisterSessionNotification` and the corresponding session-change messages where appropriate. Confirm behavior in an interactive user session. A lock event closes the current app session immediately; unlock triggers a fresh foreground query before tracking resumes.

### Sleep and resume

Handle appropriate Windows power notifications (`WM_POWERBROADCAST` / power broadcast events) through a correctly registered window/notification mechanism. Flush the current interval before suspend. On resume, reinitialize hooks/state and query foreground again. Never count time during sleep.

## 4. Session state machine

States:
- `STOPPED`
- `TRACKING_APP(app_identity)`
- `IDLE(last_foreground_app?)`
- `LOCKED`
- `SUSPENDED`
- `ERROR`

Core transitions:
- Foreground A -> B: close A; begin B if user is active and session unlocked/awake.
- Foreground A -> excluded/system/Timo: close A; do not create app usage for excluded surface.
- Active -> idle threshold crossed: close app session at idle confirmation time (`T_confirm`, forward cutoff per `ADR-002`); open `idle_intervals` starting at `T_confirm` while storing `last_input_at_utc`.
- Idle -> input activity: close idle interval; re-query foreground and begin new app session.
- Any state -> locked: close app/idle interval and enter LOCKED.
- LOCKED -> unlocked: query foreground; enter TRACKING_APP or IDLE according to last input/policy.
- Any active state -> suspend: flush and enter SUSPENDED.
- SUSPENDED -> resume: reset timing baseline, query current state, and begin a fresh interval.
- App exit/access denied: close session using last reliable monotonic time and keep partial metadata.

## 5. Time accounting

- Use a monotonic clock (for example Windows performance counter via Rust/Windows APIs or `Instant`) for elapsed durations.
- Store canonical, unsplit `usage_sessions` and `idle_intervals` with wall-clock UTC `started_at_utc`/`ended_at_utc` plus capture-time `timezone_id` and `utc_offset_minutes`, and compute `duration_ms` strictly from monotonic elapsed time (`ADR-001`).
- Never calculate duration by subtracting local wall-clock values alone; system clock adjustments or DST shifts can distort them.
- Derive local daily segments (including midnight crossings, 23h/25h DST days, and timezone travel) deterministically at query/reporting time using canonical half-open UTC boundaries (`ADR-001`). Do not split sessions at write time so write and query models are never mixed.
- No overlaps between active app sessions.
- Idle, lock, and sleep are not included in application active-use totals.
- Lock/sleep may be shown as event markers; do not needlessly persist exact sensitive context.

## 6. Debounce and filtering

Foreground hooks can emit noisy or transient changes. Use a small, evidence-based debounce/stability check (initial test window 75–150 ms, provisional) to avoid phantom sessions without losing short app switches. Ignore invalid HWNDs, Timo's own windows, known system shell surfaces when appropriate, and windows with no usable process identity. Do not broadly ignore `ApplicationFrameHost.exe`: unwrap UWP/Microsoft Store child windows via documented `EnumChildWindows` + `GetPackageFamilyName` / `GetApplicationUserModelId` (`PROCESS_QUERY_LIMITED_INFORMATION`) (`ADR-006`). If `OpenProcess` returns `ERROR_ACCESS_DENIED` on elevated or anti-cheat processes, prioritize documented Windows APIs (`CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS)` / `WTSEnumerateProcessesW`) to resolve executable basename without requesting elevation; do not make `NtQuerySystemInformation` a primary dependency (`ADR-006`).

## 7. Passive media caveat

A user may watch a video or listen to media without keyboard/mouse input. A simple idle threshold can mark this as idle even while the app remains foreground. This is a known limitation of input-based idle detection. In v1:
- Make threshold configurable.
- Explain what idle means.
- Do not claim input inactivity proves the user is not looking at the screen.
- Do not inspect media contents or browser pages.
- Consider media-session awareness only as a future, separately reviewed enhancement.

## 8. Recovery (`ADR-006`)

- Flush session changes transactionally.
- Maintain a persistent checkpoint (`last_checkpoint_utc`, `checkpoint_monotonic_ms`) updated every 30 seconds while an active session or idle interval is open, plus on every state transition.
- On startup, close stale open `usage_sessions` and `idle_intervals` rows (`ended_at_utc IS NULL`) at the last reliable checkpoint (`COALESCE(last_checkpoint_utc, started_at_utc)`) and flag `recovery_closed = 1`, never at current startup time.
- If the clock jumps, use monotonic elapsed duration and mark wall-clock anomalies for diagnostics.
- Ensure suspend/resume does not create a huge false interval.

## 9. Test matrix

- Rapid Alt+Tab between two apps.
- Same app with multiple windows.
- Background music/player remains running while another app is foreground.
- Fullscreen game switch and return.
- Game starts through Steam/Epic while launcher stays open in background.
- App closes while active.
- Idle threshold crossed exactly during active app.
- User input resumes after idle.
- Lock/unlock; sleep/resume; hibernate/resume.
- Timo window hidden/tray/quit.
- Midnight crossing and timezone adjustment.
- Process crash/access denied/PID reuse.

## 10. References

- `SetWinEventHook`: https://learn.microsoft.com/windows/win32/api/winuser/nf-winuser-setwineventhook
- `GetLastInputInfo`: https://learn.microsoft.com/windows/win32/api/winuser/nf-winuser-getlastinputinfo
- `WTSRegisterSessionNotification`: https://learn.microsoft.com/windows/win32/api/wtsapi32/nf-wtsapi32-wtsregistersessionnotification
- Windows power management: https://learn.microsoft.com/windows/win32/power/power-management

# ADR-005: Background Execution, WebView2 Lifecycle & Memory Strategy

**Date:** 2026-11-17  
**Status:** Provisional (Requires Windows 11 8 GB / 16 GB release-build benchmarking in Phase 1/3)  
**Owners:** Windows Systems Engineering, Performance Engineering  

---

## 1. Context

Timo runs continuously in the background via the Windows system tray to track foreground applications while the user works (`PF-001`, `QA-012`), while targeting provisional memory budgets of `<= 100 MB` steady-state in background/tray mode and `<= 150 MB` when the dashboard Overview is visible (`PERFORMANCE_REQUIREMENTS.md`). Tauri 2 hosts the UI inside Microsoft Edge WebView2 (`msedgewebview2.exe`), which runs multi-process browser, renderer, and GPU subprocesses.

## 2. Problem

Merely hiding a Tauri `WebviewWindow` (`window.hide()`) keeps the WebView2 renderer, GPU, and utility processes resident in RAM (often 80 to 200+ MB combined depending on loaded React/Recharts/WebGL state), risking violation of the `<= 100 MB` tray target on 8 GB Windows 11 machines (including the current 8 GB developer/test workstation). Long-hidden WebView2 windows can also exhibit blank-paint or scale-drift bugs across Windows RDP sessions or DPI transitions. Conversely, destroying the WebView2 window introduces a cold reopen delay when the user clicks "Open" in the tray.

## 3. Options Considered

1. **Option A: Hide/Show only (`window.hide()`)** — fast reopen, keeps DOM/Three.js alive unless unmounted manually, high idle RAM.
2. **Option B: Hide/Show + WebView2 `MemoryUsageLevel::Low` (`WebViewExtWindows::set_memory_usage_level`) + explicit Three.js/Canvas unmount on hide** — keeps WebView2 shell warm with reduced JS/DOM working set while freeing GPU/3D scenes.
3. **Option C: Destroy & Recreate Main `WebviewWindow` on Hide-to-Tray (`window.close()` while `RunEvent::ExitRequested` calls `api.prevent_exit()` unless user triggered Quit)** — drops WebView2 child processes to zero during tray operation so only the Rust tracking core + tray handle remain resident, recreating the window on Tray Open.

## 4. Technical Trade-offs

| Criterion | Option A: Hide Only | Option B: Hide + `MemoryUsageLevel::Low` | Option C: Destroy & Recreate on Tray |
|---|---|---|---|
| Tray steady-state RAM | High (often >100 MB) | Moderate (varies by WebView2 working-set trim) | Minimal (Rust core + SQLite + tray only; WebView2 unloaded) |
| Tray Reopen speed | Instant (<50 ms) | Fast (~50–150 ms) | Cold WebView2 init (~300–800 ms) |
| Long-idle RDP / DPI stability | Prone to blank-paint / DPI drift | Moderate (still holds long-lived HWND) | Fresh DPI/surface initialization on every reopen |
| Mascot / Recharts GPU leak risk | High unless manually torn down on visibility change | Low if visibility listener disposes WebGL | Zero residual GPU/JS heap leak while in tray |
| Tracking continuity | Continuous (if Rust-owned) | Continuous (Rust-owned) | Continuous (Rust-owned via `ExitRequested` `prevent_exit`) |
| Ephemeral UI state | Preserved in RAM | Preserved in RAM | Resets route/filter unless persisted in lightweight UI state |

## 5. Recommended Decision

Adopt a **decoupled Rust Tracking Core + Two-Tier Tray Window Strategy (Provisional pending Phase 1/3 measurements)**:

1. **100% Decoupled Native Core:** The Windows tracking engine (`platform::windows`, `TrackingCoordinator`, `RecognitionService`, SQLite writer, and `TrayIcon`) runs in Rust independently of any `WebviewWindow` existence (`PF-001`). Closing or hiding the window never pauses tracking; only explicit Tray `Pause Tracking`, Settings Pause, or Tray/Window `Quit` (`Exit`) stops tracking (`APPLICATION_STRUCTURE.md` §"Window behavior").
2. **Default Close-to-Tray Policy (`Destroy & Recreate` preferred candidate for `<= 100 MB` budget):**
   - When the user clicks window `X` with preference `Hide to tray`, persist tiny ephemeral navigation context (last route, selected date/range) and **destroy the main `WebviewWindow`**, keeping the Rust process alive via `tauri::RunEvent::ExitRequested { api, .. } => api.prevent_exit()` (when `quit_requested == false`).
   - When the user clicks `Open` from the tray, recreate the main `WebviewWindow` via `WebviewWindowBuilder`.
   - When the user clicks `Minimize` (taskbar minimize, not tray close), `hide()` or native minimize applies `MemoryUsageLevel::Low` via Tauri/WRY `WebViewExtWindows::set_memory_usage_level` and immediately disposes/pauses Three.js/R3F rendering.
3. **Benchmark-Driven Gate:** During Phase 1/3, benchmark **Option B** (`hide()` + `MemoryUsageLevel::Low` + mascot unmount) vs **Option C** (`destroy()` + recreate) across 3 runs (median + max RAM, Rust + `msedgewebview2.exe` child processes combined, and reopen latency) on the 8 GB Windows 11 target machine. Keep Option C if Option B exceeds `100 MB` steady-state tray RAM; keep Option B if it measures `<= 100 MB` without RDP/DPI blanking.
4. **Visible Dashboard Rendering Discipline:**
   - Lazy-load Three.js/R3F/Drei + mascot GLB (`React.lazy`).
   - Pause R3F `frameloop` when `document.hidden`, off-screen (`IntersectionObserver`), minimized, or `reduced_motion` is enabled (`PF-004`).
   - Virtualize `Applications` list (`TanStack Virtual`) and emit only coarse IPC events (avoid streaming raw 1s idle ticks to React).

## 6. Decision Status

**Provisional** (architectural mechanism selected; final choice between permanent `destroy()` vs `hide() + MemoryUsageLevel::Low` depends on Phase 1/3 Windows 11 memory benchmarks).

## 7. Consequences

- **Positive:** Protects the strict `<= 100 MB` tray memory budget on 8 GB Windows 11 hardware and eliminates cumulative WebView2/WebGL leaks during multi-day uptime.
- **Negative / Risk:** Window recreation adds sub-second startup latency on Tray Open and requires restoring active filter/route state cleanly from Rust/local preference state. Rapid close/open cycling must be tested against WebView2 handle leaks.

## 8. Implementation Implications

- Store user's explicit close behavior preference (`tray` vs `exit`) and never keep background tracking alive if the user explicitly selected `Exit`.
- Track an explicit `is_quitting: AtomicBool` in Rust app state so `RunEvent::ExitRequested` calls `api.prevent_exit()` **only** when transitioning to tray-only mode, and exits cleanly (unregistering `UnhookWinEvent` and flushing SQLite) on Quit.
- Never reference `app.get_webview_window("main").unwrap()` inside the tracking engine (`platform::windows` / `TrackingCoordinator`); emit UI refresh events defensively only if a main window currently exists.

## 9. Validation Requirements

1. Release-build measurement script recording Rust process + all `msedgewebview2.exe` child processes (`WorkingSet` / `PrivateBytes`) over 10-minute tray idle (`<= 100 MB` target) and visible Overview (`<= 150 MB` target), 3 trials reporting median and peak (`PERFORMANCE_REQUIREMENTS.md`).
2. 20x repeated Tray Open -> Close-to-Tray cycle verifying zero WebView2 zombie processes, zero leaked GDI/WinEvent hooks, and zero lost foreground transitions.
3. Tray `Pause/Resume Tracking` and `Quit` verification (`QA-012`, `QA-013`).

## 10. Related Documents

- [`PERFORMANCE_REQUIREMENTS.md`](../PERFORMANCE_REQUIREMENTS.md)
- [`SYSTEM_ARCHITECTURE.md`](../SYSTEM_ARCHITECTURE.md)
- [`APPLICATION_STRUCTURE.md`](../APPLICATION_STRUCTURE.md)
- [`ANIMATION_SPECIFICATION.md`](../ANIMATION_SPECIFICATION.md)
- [`QA_TESTING_PLAN.md`](../QA_TESTING_PLAN.md) (`QA-012`, `QA-013`)

# Performance & Resource Budget

## Philosophy

“Lightweight” is a measurable product requirement. Tauri and Rust are technology choices, not automatic guarantees of low RAM or CPU consumption.

## Provisional budgets

Reference test hardware must be documented (initial test matrix: Windows 11 x64, 8 GB RAM and 16 GB RAM devices, integrated graphics where available).

| Metric | Provisional target | Measurement condition |
|---|---:|---|
| Background/tray memory | <=100 MB steady-state | Release build, no dashboard visible, 10-minute idle observation |
| Visible dashboard memory | <=150 MB target | Overview loaded, no active 3D-heavy interaction |
| Idle CPU | Near 0% average | 5-minute idle window; report actual average and peaks |
| UI responsiveness | No noticeable delay for core navigation | Release build on reference hardware |
| Tracking continuity | No dropped app transitions in test suite | Controlled event test matrix |
| Mascot rendering | Paused when hidden | CPU/GPU/frame activity verified |

These targets are provisional and may be revised only with benchmark evidence and a recorded decision. Do not advertise them as guaranteed before testing.

## Performance strategies

- WinEvent-driven foreground changes; avoid scanning every process repeatedly.
- Low-frequency bounded idle sampling.
- Single serialized tracking coordinator.
- Batch/transactional SQLite writes where safe.
- Index query paths and paginate/virtualize large application lists.
- Compute chart data at query/service level; memoize expensive frontend transforms.
- Lazy-load Three.js/R3F and mascot GLB.
- Pause 3D rendering when hidden, offscreen, minimized, or reduced-motion.
- Decouple Rust tracking from WebView2 lifetime (`ADR-005`): evaluate destroying and recreating the main `WebviewWindow` on hide-to-tray vs `hide()` + `WebViewExtWindows::set_memory_usage_level(Low)` during Phase 1/3 benchmarks; never keep charts or mascot scenes mounted in tray mode.
- Keep event payloads small; do not stream every low-level system event to React.

## Benchmark procedure

1. Build release package with production settings.
2. Record device model, CPU, RAM, Windows build, display scaling, and GPU.
3. Measure process and WebView2 child-process memory separately and as a total.
4. Measure baseline after startup, idle tray, visible overview, long application list, timeline, and mascot active.
5. Measure CPU average/peak during idle and rapid app switching.
6. Repeat at least three times and report median plus maximum.
7. Compare with previous release and investigate regressions.

## Performance acceptance

A build passes only when measurements are attached to QA results. If a target is missed, identify whether the cause is WebView2, UI state, chart rendering, mascot asset/rendering, database query, or tracking loop. Do not optimize by reducing tracking correctness.

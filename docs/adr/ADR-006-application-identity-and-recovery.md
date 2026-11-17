# ADR-006: Application Identity, Elevated/UWP Resolution, Icon Caching & Crash Recovery

**Date:** 2026-11-17  
**Status:** Approved (Documented Windows API priority constraint) / Recommended (Identity keys, icon cache & crash checkpoint implementation details)  
**Owners:** Windows Systems Engineering, Application Recognition Engineering, Security Engineering  

---

## 1. Context

Timo must reliably identify foreground Win32 installers, Microsoft Store/UWP packaged apps, Steam/Epic games, portable apps, and standalone `.exe` files (`AR-001`..`AR-010`), display app icons (`APPLICATION_STRUCTURE.md` line 33), avoid PID reuse errors (`TR-002`), survive `ERROR_ACCESS_DENIED` on elevated or anti-cheat processes (`SYSTEM_ARCHITECTURE.md` line 77), and recover orphaned sessions cleanly after crashes (`TR-008`, `TRACKING_ENGINE_SPEC.md` §8).

## 2. Problem

Several critical edge cases lacked concrete engineering rules across `TRACKING_ENGINE_SPEC.md`, `APPLICATION_RECOGNITION_SPEC.md`, and `DATABASE_SCHEMA.md`:
1. **UWP Containerization:** Microsoft Store apps render inside `ApplicationFrameHost.exe`, hiding the real package PID unless child HWNDs are inspected.
2. **Elevated & Anti-Cheat Processes:** Opening handles (`OpenProcess`) to Administrator tools or kernel anti-cheat games (e.g., Vanguard, EasyAntiCheat, BattlEye) can return `ERROR_ACCESS_DENIED` when Timo runs at standard user integrity.
3. **Portable EXEs:** Using full paths for `stable_app_key` breaks continuity when a portable folder moves; using basename alone collides across unrelated `launcher.exe` / `game.exe` files.
4. **Unsafe Icon / PE Parsing:** Shell APIs or `LoadLibrary` on untrusted EXEs can load arbitrary code or shell handlers (`SECURITY.md` forbids executing discovered EXEs).
5. **Crash Recovery Checkpoints:** `DATABASE_SCHEMA.md` did not specify how the last reliable monotonic/wall-clock checkpoint is persisted for unexpected crash cleanup.

## 3. Options Considered

1. **Identity fallback on `ERROR_ACCESS_DENIED`:**
   - Option 1A: Request UAC elevation for Timo (Rejected: excessive privilege and security risk).
   - Option 1B: Use undocumented native `NtQuerySystemInformation(SystemProcessIdInformation)` as primary fallback (Rejected per owner constraint: potential future Windows compatibility risk).
   - Option 1C: **Prioritize documented Windows APIs (`CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS)` / `WTSEnumerateProcessesW`)** to retrieve executable basename when `OpenProcess(PROCESS_QUERY_LIMITED_INFORMATION)` returns `ERROR_ACCESS_DENIED`, recording partial identity cleanly without elevation.
2. **PE & Icon extraction:** `Shell`/`LoadLibrary` vs pure read-only byte parsing (`LOAD_LIBRARY_AS_DATAFILE` or pure-Rust PE `.rsrc` resource parser) into a local PNG cache.
3. **Crash checkpoint:** Periodic WAL update (`last_checkpoint_utc`) vs keeping `ended_at_utc = NULL` without heartbeat (loses duration on crash).

## 4. Technical Trade-offs

- **Documented Win32 Fallback (`CreateToolhelp32Snapshot` / `WTSEnumerateProcessesW`):** Both APIs are official documented Windows APIs that return process image basenames (`szExeFile` / `pProcessName`) across elevated/protected processes **without calling `OpenProcess`** and without depending on undocumented `ntdll` structs (`NtQuerySystemInformation`). Their trade-off is exposing only the executable basename rather than the full filesystem path when `OpenProcess` is denied, which is completely acceptable for Timo's foreground tracking and partial-evidence model (and preserves path privacy).
- **Read-Only PE Inspection:** Parsing PE `.rsrc` sections directly from read-only file bytes in Rust (or strictly data-only Win32 `GetFileVersionInfoExW` / icon extraction avoiding `DllMain` execution) upholds `SECURITY.md` rules cleanly.
- **30-Second Heartbeat Checkpoint:** Updating `last_checkpoint_utc` and `checkpoint_monotonic_ms` every 30s in WAL mode bounds crash loss to <= 30s with negligible SSD/CPU impact (~2 writes/min).

## 5. Recommended Decision

### A. Foreground Process & UWP / Elevated Resolution Order (Documented APIs First)
1. Given valid foreground `HWND`, call documented `GetWindowThreadProcessId` -> `frame_pid`.
2. Open with documented `OpenProcess(PROCESS_QUERY_LIMITED_INFORMATION, FALSE, frame_pid)` and read executable image name (`QueryFullProcessImageNameW`) + `GetProcessTimes` (`process_started_at_utc` for PID reuse protection).
3. **If process is `ApplicationFrameHost.exe`:** Enumerate child windows via documented `EnumChildWindows` until finding a child HWND whose `GetWindowThreadProcessId` returns `child_pid != frame_pid`. Re-open `child_pid` with `PROCESS_QUERY_LIMITED_INFORMATION`, then call documented `GetPackageFamilyName` and `GetApplicationUserModelId` (`appmodel.h`) to obtain Store/UWP package identity.
4. **If `OpenProcess` fails with `ERROR_ACCESS_DENIED` (elevated / anti-cheat game):**
   - Do **not** drop the foreground switch and do **not** request UAC elevation.
   - **Do NOT use `NtQuerySystemInformation` as a primary dependency.**
   - Use documented Win32 **`CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS)`** (`Process32FirstW` / `Process32NextW` matching `th32ProcessID` -> `szExeFile` basename), with documented **`WTSEnumerateProcessesW`** (`pProcessName`) as an optional secondary documented check if needed.
   - Record the active session using the resolved executable basename with partial evidence (`classification_state` reflecting available certainty, never falsely labeling an unknown elevated binary as a game without manifest/curated proof).

### B. Stable App Identity (`stable_app_key`) Precedence & Path Normalization
Derive deterministic `applications.id` / unique `stable_app_key` using this hierarchy:
1. **Packaged / Store App:** `pkg:<package_family_name>` (or `aumid:<aumid>` if PFN absent).
2. **Signed / Rich Metadata Installed App:** `meta:<normalized_publisher_slug>:<normalized_product_slug>:<lowercase_basename>` when verified PE `ProductName` + `CompanyName` or Steam/Epic `app_id` (`steam:<app_id>`, `epic:<catalog_item_id>`) are present.
3. **Standalone / Portable EXE without rich metadata:** `exe:<lowercase_basename>:<sha256_prefix_12(pe_header_timestamp_and_size_or_normalized_install_root)>` — Wait: computing a fast, bounded hash of **non-full-file PE COFF header + version metadata** (or normalized relative path if on removable/portable volume) keeps identity stable across portable folder moves without reading multi-GB game executables into memory! Store `executable_path` locally (nullable, redacted in logs and default exports).

### C. Safe Metadata & Icon Caching
- Never execute, `LoadLibrary`, or `ShellExecute` any discovered `.exe`.
- Read PE `VS_VERSION_INFO` (`ProductName`, `FileDescription`, `CompanyName`, `FileVersion`) using read-only bounded file parsing in Rust (e.g., `pelite` crate or `GetFileVersionInfoExW` without executing code).
- Extract the best-resolution app icon (up to 64x64 / 128x128 px for 150% DPI clarity) from read-only PE `.rsrc` (`RT_GROUP_ICON` / `RT_ICON`) or UWP package manifest logo assets into `%LOCALAPPDATA%\Timo\icons\<app_id_hash>.png`.
- Delete cached icons when the user deletes app metadata (`PV-005`). Fall back cleanly to a category-matched Lucide icon when icon extraction fails or access is denied.

### D. Crash Checkpoints & Stale Session Recovery
- Persist a `last_checkpoint_utc` + `checkpoint_monotonic_ms` column on the active open `usage_sessions` row (and `idle_intervals` row per `ADR-002`), updated every **30 seconds** while active (only when state is open, not on every tick) plus immediately on any state transition, lock, suspend, or clean exit.
- On startup, scan for any row with `ended_at_utc IS NULL`. If found:
  - Set `ended_at_utc = COALESCE(last_checkpoint_utc, started_at_utc)`.
  - Set `duration_ms = MAX(0, checkpoint_monotonic_ms)` (or bounded `ended_at_utc - started_at_utc` if monotonic checkpoint missing).
  - Mark `recovery_closed = 1` (diagnostic metadata) so offline downtime is **never** counted as active usage.

## 6. Decision Status

**Approved** (Documented Windows APIs constraint: prioritize documented Win32/AppModel APIs and prohibit making `NtQuerySystemInformation` a primary dependency) / **Recommended** (identity key precedence, icon caching, and 30s WAL checkpoints ready for Phase 1/2 validation).

## 7. Consequences

- **Positive:** Store apps (`ApplicationFrameHost.exe`), elevated tools, anti-cheat protected games, portable folders, and multi-DPI app icons work cleanly using 100% documented Microsoft Windows APIs without UAC prompts, unsafe EXE execution, or undocumented `ntdll` coupling. Crash losses cap at <= 30s.
- **Negative / Risk:** When `OpenProcess` is denied on an elevated/anti-cheat process, `CreateToolhelp32Snapshot` returns only the basename (`szExeFile`) rather than the full filesystem path or PE version block (mitigated by matching curated rules/manifests and allowing user review/override).

## 8. Implementation Implications

- Update `DATABASE_SCHEMA.md`: add `stable_app_key` (unique index), `icon_cache_path` (nullable, local only), `last_checkpoint_utc`, `checkpoint_monotonic_ms`, and `recovery_closed` flag.
- Update `APPLICATION_RECOGNITION_SPEC.md` and `TRACKING_ENGINE_SPEC.md` with UWP child enumeration, elevated `ERROR_ACCESS_DENIED` fallback, and icon cache deletion requirements.

## 9. Validation Requirements

1. Native Windows 11 tests covering: UWP app (`Calculator` / `Microsoft Store` app via `ApplicationFrameHost.exe`), elevated Task Manager (`ERROR_ACCESS_DENIED` fallback), Steam game + background Steam launcher (`QA-008`), portable `.exe` moved across directories, malformed/0-byte `.exe` fuzz input (`SECURITY.md`), and forced `TerminateProcess` crash recovery proving downtime is excluded.

## 10. Related Documents

- [`TRACKING_ENGINE_SPEC.md`](../TRACKING_ENGINE_SPEC.md)
- [`APPLICATION_RECOGNITION_SPEC.md`](../APPLICATION_RECOGNITION_SPEC.md)
- [`DATABASE_SCHEMA.md`](../DATABASE_SCHEMA.md)
- [`SECURITY.md`](../SECURITY.md)
- [`PRIVACY_SECURITY.md`](../PRIVACY_SECURITY.md)
- [`QA_TESTING_PLAN.md`](../QA_TESTING_PLAN.md) (`QA-008`, `QA-009`, `QA-010`, `QA-014`)

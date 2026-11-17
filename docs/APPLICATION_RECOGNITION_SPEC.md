# Application Recognition & Classification Specification

## Objective

Identify the foreground application reliably, including conventional installers, Microsoft Store/package apps, Steam/Epic games, portable applications, and standalone EXE files.

## Recognition evidence

Collect only metadata needed to identify the app:
- Executable basename and normalized path.
- Process creation identity and package/AUMID identity when available.
- PE version resource: product name, file description, company/publisher, product/file version.
- Digital-signature publisher when practical and available.
- Installed-app registry/shortcut metadata where safely accessible.
- Platform manifests such as Steam app manifests or Epic metadata when available and readable.

Do not execute the executable, load arbitrary DLLs, inspect user documents, or recursively scan the entire disk to discover apps.

## Recognition precedence

1. Explicit user override.
2. Exact package identity / curated application database match.
3. Strong product + publisher + executable identity match.
4. Launcher/platform manifest match.
5. Verified installation metadata and normalized file path.
6. Filename/path heuristic.
7. Unknown / review required.

Publisher alone is not enough to identify a specific app. A filename alone is weak evidence.

## Stable app identity & safe metadata (`ADR-006`)

Use a deterministic `stable_app_key` precedence (`pkg:<package_family_name>`/`aumid:<aumid>` > `meta:<publisher_slug>:<product_slug>:<lowercase_basename>` or Steam/Epic manifest ID > `exe:<lowercase_basename>:<pe_header_or_root_hash_12>`) rather than raw volatile paths alone so portable applications keep continuity across folder moves without colliding on generic names like `app.exe`. For UWP containers (`ApplicationFrameHost.exe`), resolve child HWNDs (`EnumChildWindows`) to extract documented `GetPackageFamilyName` / `GetApplicationUserModelId`. For elevated or anti-cheat processes returning `ERROR_ACCESS_DENIED`, prioritize documented Windows APIs (`CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS)` / `WTSEnumerateProcessesW`) to obtain the executable basename as partial identity without elevating Timo; do not rely on `NtQuerySystemInformation` as a primary dependency (`ADR-006`). Do not use `PID` as the application ID; pair `PID` + `process_started_at_utc` to avoid PID reuse collisions.

Read PE version resources and extract app icons strictly from non-executable read-only bytes (`LOAD_LIBRARY_AS_DATAFILE` or pure-Rust PE `.rsrc` parsing, never executing EXEs or shell handlers), caching normalized PNG icons under local app data (`%LOCALAPPDATA%\Timo\icons\`) and deleting them whenever user metadata is cleared.

The full executable path can expose personal folder names. Keep it local, restrict access, redact it from diagnostic exports by default, and allow metadata deletion. Avoid showing full paths in ordinary UI.

## Categories

Initial categories:
- Productivity
- Entertainment
- Gaming
- Communication
- System & Utilities
- Others / Uncategorized

Separate **application role/type** from category where needed:
- game
- game launcher
- media player
- browser
- editor/IDE
- communication client
- utility

For example, Steam can be labeled as a game launcher within Gaming, while a game executable foregrounded from Steam is tracked as its own application. Do not credit background Steam time while the game window is foreground.

## Classification confidence

Use evidence states, not a misleading probability pretending to be calibrated:
- `confirmed_user`: user explicitly selected the category.
- `high`: exact package/curated signature match.
- `medium`: multiple metadata signals agree.
- `low`: heuristic/name/path evidence only.
- `unknown`: insufficient evidence.

Store evidence source and rule ID so behavior can be explained and corrected.

## Browser ambiguity (`ADR-003`)

A browser (`chrome.exe`, `msedge.exe`, `firefox.exe`, `brave.exe`, etc.) can be used for work, school, communication, games, and entertainment. Browser URL/tab tracking is out of scope for v1 (`PV-004`). Per `ADR-003` (**Approved** owner decision), Timo retains the existing 6 MVP categories without adding a `Web & Browsing` category, recognizes known browsers cleanly (`display_name`, icon, `app_type = "browser"`), defaults their category neutrally to `Others / Uncategorized` (`classification_state = "low"` category certainty so they appear in the Review Queue without blocking tracking), never assumes browsers are Productivity apps, and persists manual user category overrides (`confirmed_user`). Never describe a browser default as an objective statement about user intent.

## Launcher and game detection

- Detect the process owning the foreground game window.
- Use launcher manifests as supporting identification evidence, not as a reason to count the launcher process.
- Support games installed outside default directories when the foreground EXE is detected.
- If a game uses a helper window owned by a launcher/broker, retain uncertainty and provide a correction path.
- Do not scan all running processes and infer game usage from background existence.

## User correction

- Every application can be assigned a category manually.
- Manual override persists across future runs.
- Provide reset-to-automatic option.
- Unknown apps continue to be tracked; classification must never block session recording.

## Security and robustness

Treat paths, file metadata, manifests, and product names as untrusted input. Normalize safely, constrain filesystem reads to metadata operations, avoid shell interpolation, and never launch a discovered EXE. Handle access denied, malformed metadata, missing files, and deleted executables gracefully.

## Test corpus

Build a labeled corpus containing browsers, office tools, creative apps, IDEs, media players, communication apps, Steam/Epic launchers, games installed through those launchers, Microsoft Store apps, portable apps, and renamed/custom EXEs. Measure precision/recall per evidence tier; unknown is an acceptable outcome when evidence is weak.

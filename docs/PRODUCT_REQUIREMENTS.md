# Product Requirements — Timo

**Purpose:** Detailed, testable behavior requirements that expand the summary in `PRD.md` for **Timo — A little clarity for your digital day.** (**APPROVED — OWNER DECISION**).

Priority labels: **P0** release-critical, **P1** important for first release, **P2** future.

## 1. Tracking requirements

| ID | Priority | Requirement | Acceptance criteria |
|---|---|---|---|
| TR-001 | P0 | Detect the foreground window on Windows. | Controlled app switches produce one ordered transition each; Timo and excluded system surfaces are filtered. |
| TR-002 | P0 | Identify the process associated with the foreground window. | Store stable app identity and session process metadata; PID reuse is not treated as the same process instance. |
| TR-003 | P0 | Start/stop app usage sessions on foreground changes. | Previous session closes at transition time; next eligible app begins without overlap. |
| TR-004 | P0 | Do not count background processes. | An app left running behind another window accrues no usage duration. |
| TR-005 | P0 | Detect idle based on last input time (`ADR-002`). | At configured threshold, active app usage closes cleanly per `ADR-002` policy boundary and an idle interval is recorded separately. |
| TR-006 | P0 | Detect lock/unlock. | Lock ends active app or idle interval immediately; unlock starts a fresh eligible session after foreground state is re-evaluated. |
| TR-007 | P0 | Detect suspend/resume. | Suspend closes/flushes current app or idle interval; resume reinitializes state and does not count sleep time. |
| TR-008 | P0 | Handle app exit and Timo shutdown (`ADR-006`). | Current session is finalized or safely recovered at last reliable checkpoint; no unbounded open intervals. |
| TR-009 | P0 | Handle local day boundaries (`ADR-001`). | Canonical UTC sessions project onto local calendar days and DST transitions at query time without duplication or loss. |
| TR-010 | P1 | Allow idle threshold configuration. | User can change threshold; new value applies predictably and is persisted locally. |
| TR-011 | P1 | Exclude Timo's own UI by default. | Dashboard interaction is not counted as ordinary application usage. |

## 2. Application recognition requirements

| ID | Priority | Requirement | Acceptance criteria |
|---|---|---|---|
| AR-001 | P0 | Build application identity from executable/package evidence (`ADR-006`). | One installed app remains consistently identified across sessions via deterministic `stable_app_key`. |
| AR-002 | P0 | Read available executable metadata (`ADR-006`). | Product name, file description, company/publisher, version, and path are read defensively without executing untrusted binaries. |
| AR-003 | P0 | Support standalone EXE applications (`ADR-006`). | An executable launched directly can be recorded even when no installer/launcher metadata exists. |
| AR-004 | P0 | Distinguish launchers from launched apps/games. | Foreground game process is tracked; background Steam/Epic launcher is not credited with game time. |
| AR-005 | P1 | Recognize packaged Windows apps (`ADR-006`). | Resolve `ApplicationFrameHost.exe` child HWNDs and read Package Family Name / AUMID evidence when accessible. |
| AR-006 | P1 | Use platform manifests when available. | Steam/Epic identification enriches metadata without requiring launcher process time to be counted. |
| AR-007 | P0 | Categorize with evidence and confidence (`ADR-003`). | Rule provenance and confidence state are available for user review; known browsers use neutral default category rather than assuming Productivity. |
| AR-008 | P0 | Support manual category correction. | User override takes precedence over future automatic rules. |
| AR-009 | P0 | Provide unknown/review state. | Low-confidence apps are not presented as definitively classified. |
| AR-010 | P1 | Merge app instances safely. | Multiple windows/process instances of the same app aggregate to the intended application record while session boundaries remain correct. |

## 3. Analytics and interface requirements

| ID | Priority | Requirement | Acceptance criteria |
|---|---|---|---|
| UI-001 | P0 | Overview shows today's active use. | Sum equals valid active application intervals for the selected local day. |
| UI-002 | P0 | Overview shows top applications and categories. | Ordering and totals match database queries. |
| UI-003 | P0 | Application list supports search, category filter, and sorting. | Results update correctly without loading every row into the DOM. |
| UI-004 | P0 | Application detail shows daily and weekly duration. | Date range and timezone behavior are explicit and consistent. |
| UI-005 | P0 | Application detail shows average daily use and last used. | Values are computed over a clearly defined date range and exclude days with no data only according to documented metric rules. |
| UI-006 | P0 | Timeline shows chronological activity. | App intervals do not overlap; idle/lock/sleep markers are distinguishable from usage. |
| UI-007 | P1 | Insights show weekly/monthly trends. | Aggregates reconcile with session-level data. |
| UI-008 | P0 | Unknown apps can be reviewed. | User can identify, categorize, or leave unknown without blocking tracking. |
| UI-009 | P1 | Greeting uses locally suggested Windows username. | User confirms/edits name; neutral fallback exists; no remote transmission. |
| UI-010 | P1 | Light/dark themes and reduced motion (`ADR-004`). | Semantic light/dark tokens pass WCAG 2.2 AA contrast (`4.5:1` normal text, `3.0:1` large text and UI boundaries) and reduced motion disables nonessential transitions. |

## 4. Privacy and data requirements

| ID | Priority | Requirement | Acceptance criteria |
|---|---|---|---|
| PV-001 | P0 | No account required. | Fresh install can complete onboarding offline. |
| PV-002 | P0 | Usage data remains local. | No usage history or executable metadata is transmitted by default. |
| PV-003 | P0 | No keystroke or screenshot capture. | Code and runtime permissions contain no such feature. |
| PV-004 | P0 | No browser URL/tab collection in v1. | Browser is tracked only as an application. |
| PV-005 | P0 | User can delete local usage data. | Delete action removes selected/all usage records and confirms destructive action. |
| PV-006 | P1 | User can export records. | Export includes clear schema, time zone, and privacy warning. |

## 5. Performance requirements

| ID | Priority | Requirement | Acceptance criteria |
|---|---|---|---|
| PF-001 | P0 | Tracking continues with dashboard hidden/tray (`ADR-005`). | Session collection remains operational in Rust without continuous UI rendering or active WebView2 frames. |
| PF-002 | P0 | Foreground tracking is event-driven. | No aggressive full-process scan loop is used for each time tick. |
| PF-003 | P0 | Idle sampling is low-frequency and bounded. | Sampling interval is documented and benchmarked. |
| PF-004 | P1 | Mascot rendering pauses when not visible. | No continuous animation/render loop when hidden, minimized to tray, or off-screen. |
| PF-005 | P0 | Meet provisional memory budget. | Release-build measurements and hardware are recorded in QA report. |

## 6. Acceptance principles

- Totals must reconcile: app totals + idle totals are distinct; locked and sleep time are excluded from active use.
- Avoid false precision: label estimated/unknown classifications honestly.
- No undocumented background network activity.
- Every user-facing metric has a written formula.
- Every feature has loading, empty, error, and disabled states where applicable.

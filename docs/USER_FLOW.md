# User Flow — Timo

## 1. First launch / onboarding

1. **Welcome:** introduce Timo and the core benefit in one short screen.
2. **Privacy explanation:** explain foreground app detection, local storage, what is not collected, and how to pause/stop tracking.
3. **Personal greeting:** read the current Windows account name locally as a suggestion; allow confirmation/editing. If unavailable, ask for an optional nickname or use a neutral greeting. Do not force a name.
4. **Tracking preferences:** show idle threshold (recommended initial value: 5 minutes, explicitly configurable), launch-at-login opt-in, and whether Timo should start minimized to tray.
5. **System readiness:** verify database can be created, required Windows hooks can be registered, and the current session is interactive. Explain any permissions or limitations.
6. **Start tracking:** initialize the tracking engine and show a clear tracking status.
7. **Overview:** show an empty/initial state explaining that insights appear as data is collected; do not fabricate example statistics as real data.

## 2. Returning user

- Launch or restore Timo from the tray.
- Overview greets with the saved display name when available.
- Show current day, tracked active time, idle time separately, top apps, and category distribution.
- Provide a visible tracking state and last update timestamp.

## 3. Application review flow

1. Recognition engine encounters an unknown or low-confidence foreground app.
2. The app is recorded under Others/Uncategorized; tracking must not be blocked.
3. Review center presents best available identity evidence: app name, executable name, publisher (when present), and source.
4. User can choose a category or leave it unknown.
5. Save a persistent user override.
6. Future sessions use the override until the user changes it.

## 4. Application detail flow

1. User opens Applications.
2. Search/filter/sort the list.
3. Select an app.
4. Application Details displays today, selected period, weekly use, average per day, last used, category, and a chronological session history.
5. User can edit category and optionally exclude the application from tracking.

## 5. Timeline flow

- Choose Today or a date.
- Inspect a chronological vertical or horizontal timeline.
- App-active intervals use the application's category accent.
- Idle intervals are visually distinct and never counted as app time.
- Lock/sleep are interruption markers, not active usage blocks.
- Selecting an app interval opens its application details.

## 6. Insights flow

- Select week/month period.
- Review total active app time, category split, top apps, and trends.
- Compare periods only when both periods contain valid data.
- Explain metric definitions in tooltips/help text.
- Avoid value judgments such as "bad day" or "wasted time".

## 7. Settings flow

Groups:
- General: greeting name, launch at login, tray behavior.
- Tracking: idle threshold, pause/resume state, excluded apps.
- Categories: category overrides and reset-to-suggested option.
- Appearance: light/dark/system, text scaling if supported, reduced motion.
- Privacy & Data: export, delete data, local database location information.
- About: version, diagnostics, licenses, privacy summary.

## 8. Failure and recovery flows

- **Tracking unavailable:** explain the reason; show no fake activity totals; provide retry.
- **Database locked/corrupt:** preserve a recoverable backup where possible; do not silently overwrite; provide diagnostic export with sensitive paths redacted.
- **Unknown app:** continue tracking under uncategorized.
- **Mascot asset unavailable:** core application remains usable; show a graceful non-mascot fallback only as a documented degraded state, not as a claim that 3D production succeeded.
- **Network unavailable:** normal tracking and analytics continue; remote asset-generation tools are not part of app runtime.

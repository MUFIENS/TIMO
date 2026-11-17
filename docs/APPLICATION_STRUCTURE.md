# Application Structure & Information Architecture

## Navigation model

Use a persistent, compact desktop sidebar with five primary destinations:

1. **Overview** — today at a glance.
2. **Timeline** — chronological activity.
3. **Applications** — recognized apps, categories, and review queue.
4. **Insights** — weekly/monthly analytics.
5. **Settings** — preferences and data controls.

Application Details is a nested route/panel from Applications and relevant dashboard/timeline items. It is not a sixth primary navigation item.

## Page responsibilities

### Overview
- Personalized greeting, with a user-confirmed local display name.
- Today's active application usage (primary metric).
- Idle time presented separately.
- Category summary, top apps, short timeline preview, tracking status.
- Never use mock/demo data in a live empty database.

### Timeline
- Day selector and chronological activity lanes/segments.
- Filter by category/application.
- App-active blocks, idle blocks, and lock/sleep interruption markers.
- Selecting a block opens app details.
- Handle long days, gaps, and sessions crossing midnight.

### Applications
- Search, category filters, sort by usage/name/last used.
- List or table with icon, app name, category, duration, last used.
- Unknown/review state.
- App metadata drawer or detail route.
- Virtualize large lists.

### Application Details
- App identity and executable/product metadata when available.
- Category and source/confidence explanation.
- Today, week, selected date range, average daily usage, last used.
- Usage history/timeline.
- Edit category, exclude/include tracking, reset classification override.

### Insights
- Daily/weekly/monthly period selector.
- Active time trend, category distribution, top apps, day-of-week patterns.
- Clear date/timezone labels and metric definitions.
- Do not infer productivity from usage duration alone.

### Settings
- General and greeting name.
- Tracking and idle threshold.
- Startup/tray.
- Categories and exclusions.
- Appearance/accessibility.
- Privacy, export, deletion.
- About and diagnostics.

## Global UI elements

- Sidebar brand mark and mascot identity (mascot presence may be contextual, not omnipresent).
- Current tracking state indicator with text, not color alone.
- Date range context.
- Toasts for completed user actions; dialogs for destructive actions.
- Empty/loading/error states for every data-heavy surface.

## Window behavior

- Minimum window size must be selected after layout testing; recommended initial design target 980×640 logical pixels, with graceful behavior below it.
- Support resizing and Windows display scaling.
- Closing the main window should follow explicit user preference: hide to tray or exit. Never silently keep tracking if the user explicitly chose Exit.
- System tray menu must offer Open, Pause/Resume Tracking, and Quit.

# Watch App

## Overview
A standalone watchOS companion app that displays today's steps, weekly and monthly step summaries, and recent workouts. Data is sourced from both HealthKit directly on the watch and synced from the iPhone app via WatchConnectivity. Complications are supported via WidgetKit.

## User Experience
The app uses a vertical paging `TabView` with four tabs navigated by scrolling up/down:

1. **Today tab** - Shows today's step count in large serif font, distance and calories in secondary style, and (if goal enabled) a horizontal progress bar with percentage. The background gradient color reflects goal progress. Current day and weekday are shown in the top-left corner of the background.

2. **This Week tab** - Shows total steps for the week (or average steps/day if the user set "average" display mode) in large serif font, and a 7-bar `Chart` with weekday labels. Goal-achieved bars are `goalCompleted` gradient, below-goal bars are `goalInProgress`, future days are faint gray. A dashed `RuleMark` shows the step goal line when enabled.

3. **Month tab** - Shows total steps for the month and a compact `CalendarView` heat-map (the same shared component used in the Monthly Calendar widget) without a header, with spacing of 6pt. Background uses a purple gradient.

4. **Workouts tab** - Scrollable list of recent workouts (up to 5, synced from iPhone). Each row shows the workout icon, activity name, duration, and relative date ("Today", "Yesterday", or abbreviated day + date). Empty state shows a running figure icon.

App refreshes data on every `WKApplication.didBecomeActiveNotification` and when the calendar day changes (`NSCalendarDayChanged`).

## Key Components
- **`StepsWatchApp`** — `@main` entry point. Injects `HealthKitManager` and `WatchConnectivityManager.shared` as `@Environment` objects into `ContentView`.
- **`ContentView`** — `TabView` with `.verticalPage` style, four tabs. Calls `fetchData()` on each tab's `.task` and listens for active/day-change notifications.
- **`TodayView`** — Reads `Defaults[.watchCurrentSteps/Distance/Calories]`. Shows `ProgressBar` (custom capsule bar with percentage label) when goal enabled.
- **`WeekView`** — Reads `connectivityManager.weeklyRecords` (synced from iPhone). Builds `WeekViewData` with per-day historical goals. Supports `stepCountMode` toggle between total and average display.
- **`MonthView`** — Reads `connectivityManager.monthlyRecords`. Computes `dayProgresses: [Int: Double]` using same logic as the widget (goal-relative or max-relative).
- **`WorkoutsView`** — Reads `connectivityManager.recentWorkouts`. `WorkoutRowView` formats duration and relative date.
- **`WatchStreakView`** — Separate view (not in main tab flow but available) showing streak count, 7-day achievement row, and today's progress gauge — mirrors the Streak widget UI on the watch.
- **`ProgressBar`** — Custom capsule track + fill bar with percentage text overlay, animated.
- **`WeeklyMiniChart`** — Compact HStack of small bar columns, used for lightweight weekly previews.
- **`CalendarView`** — Shared with widget extension; renders month grid with day progress heat-map.
- **`WatchStepRecord`** — Codable struct for synced daily records: `id`, `date`, `stepCount`, `distance`, `calories`, `flightsClimbed`. Decodes with optional fallbacks for backward compatibility.

## Data & Logic
- **HealthKit (direct)**: `HealthKitManager` requests read access for `stepCount`, `distanceWalkingRunning`, `activeEnergyBurned`. Fetches today's cumulative sums via `HKStatisticsQuery`. Results stored in `Defaults[.watchCurrentSteps/Distance/Calories]` (shared App Group).
- **Background delivery**: `HKObserverQuery` on `stepCount` with `.immediate` frequency triggers `fetchHealthData()` whenever new step data arrives, even when the app is backgrounded.
- **Day reset**: On `fetchHealthData()`, if the last update timestamp is from a different calendar day, watch-side step defaults are reset to 0 and complications reloaded.
- **WatchConnectivity sync**: `WatchConnectivityManager` (watch side) receives weekly records, monthly records, recent workouts, step goal settings, unit system, week start, step count mode, streak count, and goal change log from the iPhone via `sendMessage` or `applicationContext`.
- **Complication reload**: `HealthKitManager.reloadComplicationsThrottled()` — throttled to at most once per 30 seconds — calls `WidgetCenter.shared.reloadAllTimelines()` to update watch complications.
- **Today record sync**: After each HealthKit fetch, `WatchConnectivityManager.updateTodayRecord(steps:distance:calories:)` updates the in-memory `weeklyRecords` and `monthlyRecords` arrays and persists them to `Defaults`.
- **Historical goals**: `Defaults[.goalChangeLog]` is synced from iPhone and used by `getGoalForDate(_:goalLog:)` to color bars correctly even if the goal changed during the week/month.

## Premium
No Pro gating on the watch app. All four tabs are available to all users. Streak view requires `enableDailyStepGoal` to be meaningful.

## Related Files
- `/Users/hieudinh/Projects/Steps/StepsWatch Watch App/StepsWatchApp.swift`
- `/Users/hieudinh/Projects/Steps/StepsWatch Watch App/ContentView.swift`
- `/Users/hieudinh/Projects/Steps/StepsWatch Watch App/TodayView.swift`
- `/Users/hieudinh/Projects/Steps/StepsWatch Watch App/WeekView.swift`
- `/Users/hieudinh/Projects/Steps/StepsWatch Watch App/MonthView.swift`
- `/Users/hieudinh/Projects/Steps/StepsWatch Watch App/WorkoutsView.swift`
- `/Users/hieudinh/Projects/Steps/StepsWatch Watch App/WatchStreakView.swift`
- `/Users/hieudinh/Projects/Steps/StepsWatch Watch App/WatchStepRecord.swift`
- `/Users/hieudinh/Projects/Steps/StepsWatch Watch App/HealthKitManager.swift`
- `/Users/hieudinh/Projects/Steps/StepsWatch Watch App/WatchConnectivityManager.swift`

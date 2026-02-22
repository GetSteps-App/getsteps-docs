# Watch Connectivity

## Overview
Bidirectional data sync between the Steps iPhone app and the watchOS companion app via the `WatchConnectivity` framework. The iOS-side `WatchConnectivityManager` pushes step data, settings, and workout records to the watch; the watch-side `WatchConnectivityManager` receives them, caches them locally, and keeps complications up to date.

## User Experience
- Transparent to the user — data syncs automatically whenever the watch is paired, the watch app is installed, and connectivity is available.
- When the watch app becomes active, it requests a fresh data sync from the iPhone immediately.
- If the iPhone app is unreachable (watch not in range, Bluetooth off), the last received values are served from local cache (App Group `Defaults`).
- When settings change on the iPhone (step goal, unit system, week start, step count mode), the watch reflects the changes within seconds if reachable, or on next connection via `applicationContext`.
- Complication data (step count, distance, calories) is pushed via `transferCurrentComplicationUserInfo` to update watch face complications even when the watch app is not in the foreground.

## Key Components

### iOS Side (`Steps/Utilities/WatchConnectivityManager.swift`)
- **`WatchConnectivityManager`** (iOS) — `@Observable NSObject` singleton. Implements `WCSessionDelegate`.
- **`sendCurrentStepsToWatch(_:currentDistance:currentCalories:)`** — Sends live step metrics. Uses `sendMessage` if reachable, `updateApplicationContext` otherwise. Also fires `transferCurrentComplicationUserInfo` if complications are enabled.
- **`sendStepDataToWatch(...)`** — Full sync: current steps + distance + calories + weekly records + monthly records + streak count + goal change log. Strips `hourlySteps` from records to reduce payload size. Sends complication-only update separately (without weekly/monthly arrays).
- **`sendStepGoalToWatch(_:enableDailyStepGoal:unitSystem:weekStartsOn:stepCountMode:)`** — Sends settings changes. Queues message for retry if session not yet activated.
- **`sendInitialDataToWatch()`** — Called when session activates. Sends step goal settings + notification settings.
- **`sendNotificationSettingsToWatch()`** — Sends `enableNotifications` bool to watch.
- **`sendWorkoutsToWatch(_:)`** — Sends the 5 most recent workouts as JSON-encoded `[WorkoutData]`.
- **Retry mechanism** — Failed `sendMessage` calls retry up to 3 times with exponential backoff (1s, 2s, 4s). After max retries, falls back to `updateApplicationContext`.
- **Connectivity validation** — `validateConnectivity()` caches result for 5 seconds to avoid redundant checks. Requires `activated`, `isPaired`, `isWatchAppInstalled`.
- **Incoming messages** — Responds to `requestInitialData: true` from the watch by calling `sendInitialDataToWatch()`.

### watchOS Side (`StepsWatch Watch App/WatchConnectivityManager.swift`)
- **`WatchConnectivityManager`** (watch) — `@Observable NSObject` singleton. Implements `WCSessionDelegate`.
- **Published properties** — `stepGoal`, `enableDailyStepGoal`, `unitSystem`, `weekStartsOn`, `stepCountMode`, `enableNotifications`, `currentSteps`, `currentStreakCount`, `weeklyRecords`, `monthlyRecords`, `recentWorkouts`.
- **`requestInitialData()`** — Sends `requestInitialData: true` to iPhone when session activates or app becomes active. Queues request for retry if session not activated. Retries up to 3 times with exponential backoff.
- **`updateTodayRecord(steps:distance:calories:)`** — Called by `HealthKitManager` after each HealthKit fetch. Upserts today's record in both `weeklyRecords` and `monthlyRecords`, then re-encodes and persists to `Defaults`.
- **Incoming message handling** — All delegate methods (`didReceiveMessage`, `didReceiveApplicationContext`, `didReceiveComplicationUserInfo`) funnel through the same set of `updateX(from:)` helpers:
  - `updateStepGoalSettings(from:)` — Merges `stepGoal`, `enableDailyStepGoal`, `unitSystem`, `weekStartsOn`, `stepCountMode`, `enableNotifications`, `goalChangeLog`.
  - `updateCurrentStepsIfGreater(from:)` — Only updates if received value exceeds current (prevents rollback from stale messages).
  - `updateCurrentStreakCount(from:)` — Stores streak count to `Defaults`.
  - `updateWeeklyMonthlyRecords(from:)` — Decodes `[WatchStepRecord]` from JSON `Data`, sorts by date, persists.
  - `updateRecentWorkouts(from:)` — Decodes `[WorkoutData]` from JSON `Data`, sorts by `startDate` descending.
- **`updateComplicationsIfNeeded()`** — Calls `HealthKitManager.reloadComplicationsThrottled()` after settings or step count changes.
- **Local cache** — On init, `loadCachedRecords()` restores `weeklyRecords`, `monthlyRecords`, and `recentWorkouts` from `Defaults` so the UI is populated immediately without waiting for a sync.

## Data & Logic
- Transport format: dictionary `[String: Any]` with typed values. Records encoded as `Data` via `JSONEncoder`/`JSONDecoder`.
- Payload keys: `currentSteps`, `currentDistance`, `currentCalories`, `stepGoal`, `enableDailyStepGoal`, `unitSystem`, `weekStartsOn`, `stepCountMode`, `enableNotifications`, `goalChangeLog`, `weeklyRecords`, `monthlyRecords`, `recentWorkouts`, `currentStreakCount`, `timestamp`.
- Complication payload is a smaller subset (steps, distance, calories only) to stay within the `transferCurrentComplicationUserInfo` size budget.
- `hourlySteps` stripped from records before encoding to reduce payload — the watch has no use for per-hour data.
- `unitSystem` is migrated from legacy `useKilometers: Bool` format via the watch-side `updateStepGoalSettings(from:)` if the old key is present.
- `goalChangeLog` sent as JSON `Data` so the watch can resolve historical goals per day (for chart bar coloring).
- Session reactivation: on `sessionDidDeactivate` (iOS), the session is immediately reactivated to support Apple Watch switching.

## Premium
No gating — watch connectivity works for all users. The data synced reflects whatever the user has access to on the iPhone side.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Utilities/WatchConnectivityManager.swift`
- `/Users/hieudinh/Projects/Steps/StepsWatch Watch App/WatchConnectivityManager.swift`
- `/Users/hieudinh/Projects/Steps/StepsWatch Watch App/WatchStepRecord.swift`
- `/Users/hieudinh/Projects/Steps/StepsWatch Watch App/HealthKitManager.swift`

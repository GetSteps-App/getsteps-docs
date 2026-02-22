# Data Models

## Overview
The core data layer of the Steps app. `DataStore` is the central `@Observable` singleton that owns all step records, fetches data from HealthKit, maintains caches, and computes derived values. Supporting model types describe individual day records, hourly data, workout sessions, weather conditions, and encouraging message keys.

---

## DataStore

**File:** `DataStore.swift`, `DataStore+Mocks.swift`

### Responsibilities
- Owns `stepDailyRecords: [StepDailyRecord]` — the full historical record array, persisted via `Defaults[.stepDailyRecords]`.
- Owns `workouts: [WorkoutData]` — HealthKit workout sessions, persisted via `Defaults[.workouts]`.
- Exposes current-day live values: `stepCount`, `distance` (km), `calories` (kcal), `flightsClimbed`, `message`, `stepProgress`, `welcomeMessage`.
- Caches records by month (`monthlyRecordsCache: [String: [StepDailyRecord]]`, key format `"YYYY-MM"`) and by week (`weeklyRecordsCache: [String: [StepDailyRecord]]`, key format `"YYYY-MM-dd"` of week start).
- Tracks `currentStreakCount: Int` — recalculated on init and after every `stepDailyRecords` change.

### Key Methods
- `fetchStepsForDate(_:completion:)` — main fetch used by the UI; updates live properties and triggers widget reload.
- `fetchStepsForDateWithoutUpdateUI(_:force:completion:)` — background/silent fetch used by background tasks and batch fills.
- `fetchMissingStepRecordsIfNeeded(completion:)` — recursively fills gaps from Jan 1 2025 (or 12 months ago) to today, skipping days already cached with non-zero steps.
- `getDailyStepsForWeek(containing:) -> [StepDailyRecord]` — uses `weeklyRecordsCache` for O(1) lookup.
- `getDailyStepsForMonth(containing:) -> [StepDailyRecord]` — uses `monthlyRecordsCache` for O(1) lookup.
- `getStepDailyRecord(for:) -> StepDailyRecord` — returns matching record or a zero-step placeholder.
- `getHourlySteps(for:) -> [HourlyStepData]` — returns `hourlySteps` from the matching daily record.
- `averageFullDaySteps() -> Int?` — mean of all non-zero daily records.
- `averageWeeklySteps() -> Int?` — mean weekly total across all weeks present in records.
- `averageMonthlySteps() -> Int?` — mean monthly total across all months present.
- `recalculateCurrentStreak()` — walks `stepDailyRecords` backward from yesterday counting consecutive goal-met days; adds 1 if today's goal is met.
- `updateWelcomeMessage()` — sets `welcomeMessage` based on current hour: "Good morning" (5–11), "Good afternoon" (12–16), "Good evening" (17–20), "Good night" (otherwise).
- `requestAuthorization()` — requests HealthKit read permissions for stepCount, distanceWalkingRunning, activeEnergyBurned, flightsClimbed, and all workout types.

### HealthKit Queries (per fetch)
Each `fetchStepsForDateWithoutUpdateUI` call fires 5 parallel `HKStatisticsQuery` / `HKStatisticsCollectionQuery` requests:
1. Step count (cumulative sum, day interval)
2. Distance walking/running (cumulative sum, day interval) → stored in km
3. Active energy burned (cumulative sum, day interval) → stored in kcal
4. Flights climbed (cumulative sum, day interval)
5. Hourly steps (cumulative sum, 1-hour intervals) → `[HourlyStepData]`

### Cache Rebuild
- `rebuildAllCachesDebounced()` — called on every `stepDailyRecords` didSet; debounces with 1-second delay via `DispatchWorkItem`.
- `rebuildAllCaches()` — runs on a background `DispatchQueue` (`com.steps.recordsCache`, `.utility` QoS); groups records by month and week keys, sorts each group ascending by date, then publishes back on main thread.

### Goal Change Log
- `Defaults[.goalChangeLog]: [GoalChangeEntry]` — ordered log of goal changes with date strings and goal values.
- `migrateGoalChangeLogIfNeeded()` — on first run, creates an initial entry at `.distantPast` with the current goal so all historical days evaluate correctly.
- `getGoalForDate(_:goalLog:)` — walks the log to find the last entry on or before the given date.

### Notification Tracking
- Tracks last goal-achievement notification date (`Defaults[.goalNotificationLastSentDate]`) and last half-goal notification date to avoid duplicate notifications per day.

### Mock Data (DataStore+Mocks.swift)
- `mockData()` — generates realistic step records from Jan 1 of the current year to today with per-weekday step count distributions and realistic hourly distributions. Also generates correlated mock workout data.

---

## StepDailyRecord

**File:** `StepDailyRecord.swift`

```swift
struct StepDailyRecord: Codable, Identifiable, Equatable, Defaults.Serializable
```

| Field | Type | Description |
|---|---|---|
| `id` | `UUID` | Unique identifier |
| `date` | `Date` | The calendar day |
| `stepCount` | `Int` | Total steps for the day |
| `distance` | `Double` | Distance in kilometers |
| `calories` | `Double` | Active calories burned (kcal) |
| `flightsClimbed` | `Int` | Flights of stairs climbed |
| `messageKey` | `EncouragingMessageKey` | Key for the motivational message |
| `hourlySteps` | `[HourlyStepData]` | 24-slot hourly step breakdown |

- `encouragingMessage: String` — computed property; resolves `messageKey` to a localized string via `messageKey.localizedMessage(stepGoal:extraSteps:remaining:)`.
- Supports migration from legacy format where messages were stored as raw strings (`legacyEncouragingMessage` coding key). Migration picks the appropriate key based on step count relative to goal.

---

## HourlyStepData

**File:** `HourlyStepData.swift`

```swift
struct HourlyStepData: Identifiable, Equatable, Codable, Defaults.Serializable
```

| Field | Type | Description |
|---|---|---|
| `hour` | `Int` | Hour of day (0–23) |
| `steps` | `Int` | Step count for that hour |

`id` is computed as `String(hour)`.

---

## WorkoutData

**File:** `WorkoutData.swift`

```swift
struct WorkoutData: Identifiable, Equatable, Codable, Defaults.Serializable
```

| Field | Type | Description |
|---|---|---|
| `id` | `UUID` | Unique identifier |
| `workoutType` | `String` | Type name (e.g., "Running", "Cycling") |
| `startDate` / `endDate` | `Date` | Session time bounds |
| `duration` | `TimeInterval` | Duration in seconds |
| `distance` | `Double` | Distance in meters |
| `activeEngery` / `totalEngery` | `Double` | Kilocalories (note: field name has typo in source) |
| `averageHeartRate` / `maxHeartRate` | `Double?` | Optional BPM values |
| `averagePace` | `Double?` | Min/km |
| `elevationGain` | `Double?` | Meters |
| `isIndoor` | `Bool?` | Indoor vs. outdoor variant |
| `temperature` | `Double?` | Celsius |
| `humidity` | `Double?` | Percentage 0–100 |

- `workoutIcon: String` — SF Symbol name via `workoutSymbolName(from:isIndoor:)`. Distinguishes treadmill/track variants on iOS 18+.
- `activityDisplayName: String` — localized display name via `workoutName(from:)`.
- `workoutColor: Color` — per-type color (e.g., Running → purple, Cycling → green, Swimming → cyan).
- `isDisplayedInActivity: Bool` — true for Walking, Running, Cycling, Hiking only.
- `shouldShowIndoorOutdoor: Bool` — true for Walking, Running, Cycling, Swimming, Rowing.

Supported workout types: Walking, Running, Cycling, Swimming, Hiking, HIIT, Traditional Strength Training, Yoga, Pilates, Elliptical, Rowing, Dance, Badminton, Cooldown, Core Training, Functional Strength Training, Jump Rope, Mixed Cardio, Pickleball, Soccer, Surfing, Table Tennis.

---

## WeatherCondition (Extension)

**File:** `WeatherCondition.swift`

Extends `WeatherKit.WeatherCondition` with:
- `symbolName: String` — SF Symbol name for each condition.
- `color(for colorScheme:) -> Color` — semantic color per condition (e.g., clear → orange, rain → blue, snow → white in dark mode).
- `emoji: String` — Unicode emoji per condition (e.g., clear → "☀️", rain → "🌧️").
- `shortDescription: String` — localized short label (e.g., "Mostly Cloudy"). Uses `String(localized:defaultValue:)` with weather-prefixed keys.

Covers ~30 distinct `WeatherCondition` cases.

---

## EncouragingMessageKey

**File:** `EncouragingMessageKey.swift`

```swift
enum EncouragingMessageKey: String, Codable, Equatable, Defaults.Serializable
```

An enum with ~200 cases covering all message variants. Cases are grouped by:

| Group | Condition | Example case |
|---|---|---|
| Daily, goal-based | <25% of goal | `dailyJourneyStarts` |
| Daily, goal-based | 25–50% | `dailyHalfwayToGoal` |
| Daily, goal-based | 50–75% | `dailySeventyFivePercent` |
| Daily, goal-based | 75–100% (with remaining) | `dailySoCloseRemaining` |
| Daily, goal-based | 100–150% | `dailyGoalCrushed` |
| Daily, goal-based | 150%+ (with extra) | `dailyLegendaryExtra` |
| Daily, no goal | <3 000 steps | `dailyNoGoalShortWalk` |
| Daily, no goal | 3 000–7 000 | `dailyNoGoalNotBad` |
| Daily, no goal | 7 000–10 000 | `dailyNoGoalCrushingIt` |
| Daily, no goal | 10 000–15 000 | `dailyNoGoalStepLegend` |
| Daily, no goal | 15 000–20 000 | `dailyNoGoalUltraWalker` |
| Daily, no goal | 20 000+ | `dailyNoGoal20KGod` |
| Weekly, goal-based | 6 tiers | `weeklyGoalCrushed` |
| Weekly, no goal | 5 tiers | `weeklyNoGoalLegend` |
| Monthly, goal-based | 6 tiers | `monthlyGoalCrushed` |
| Monthly, no goal | 5 tiers | `monthlyNoGoalLegend` |
| Empty/default | — | `empty` |

`localizedMessage(stepGoal:extraSteps:remaining:)` resolves each case to a `String(localized:comment:)` call, interpolating formatted numbers where relevant (step goal, extra steps, remaining steps).

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Models/DataStore.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/DataStore+Mocks.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/StepDailyRecord.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/HourlyStepData.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/WorkoutData.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/WeatherCondition.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/EncouragingMessageKey.swift`

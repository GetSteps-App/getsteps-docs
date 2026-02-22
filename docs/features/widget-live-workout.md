# Widget: Live Workout (Live Activity)

## Overview
A Live Activity and Dynamic Island integration that shows real-time workout metrics while a workout session is in progress. Requires iOS 26+. Updates continuously as the workout progresses and shows a completion checkmark when the session ends.

## User Experience
- **Lock Screen / StandBy banner**: Shows the workout icon and name, an elapsed time counter, and a row of three metrics — Distance, Calories, and either Pace (most workouts) or Heart Rate (hiking).
- **Dynamic Island expanded**: Leading region shows the workout icon in the workout's accent color; trailing region shows the elapsed time; bottom region shows the same Distance / Calories / Pace row.
- **Dynamic Island compact leading**: Workout icon in accent color.
- **Dynamic Island compact trailing**: Cycles every 30 seconds through three views — calories (kcal), distance (value + unit), and elapsed time — with a push-from-bottom transition animation.
- **Dynamic Island minimal**: Workout icon only.
- A green `checkmark.circle.fill` icon appears next to the workout name when `isFinished` is true.
- All numeric values animate with `.numericText` content transitions.

## Key Components
- **`WorkoutWidgetLiveActivity`** — `Widget` conforming to `ActivityConfiguration<WorkoutWidgetAttributes>`. Requires iOS 26.
- **`WorkoutWidgetAttributes`** — `ActivityAttributes` struct. Static attributes: `symbol` (SF Symbol name), `workoutActivityType` (HKWorkoutActivityType raw value), `workoutName`. Computed `workoutColor` based on activity type.
- **`WorkoutWidgetAttributes.ContentState`** — Dynamic state: `metrics: MetricsModel`, `isFinished: Bool`.
- **`MetricsModel`** — Holds `elapsedTime`, `heartRate?`, `activeEnergy?`, `distance?`, `speed?`, `speedHistory`, `predictedSpeedHistory`. Provides `distanceDisplay` (value + unit strings) and `paceDisplay` (formatted string).
- **`LiveActivityView`** — The lock screen banner view. Switches between pace and heart rate for the third metric based on `hkWorkoutActivityType == .hiking`.
- **`ElapsedTimeView`** — Dedicated elapsed time display (not shown in source but referenced; uses `TimerInterval` or similar).
- **`SpeedChart`** — Custom `GeometryReader`-based path drawing that renders actual speed history as a smooth bezier curve and predicted speed as a dashed line. Not currently surfaced in the Live Activity presentation but defined for future use.
- **Workout color mapping** — running → `.purple`, walking → `.blue`, cycling → `.green`, hiking → `.mint`, others → `Color.urange`.
- **Compact trailing cycle** — `TimelineView(.periodic(from:by:30))` rotates `cycle = Int(timestamp / 30) % 3` between calories, distance, and elapsed time.

## Data & Logic
- Live Activities are started and updated from the main app's workout session manager (not in this file) via `Activity<WorkoutWidgetAttributes>.request(...)` and `.update(...)`.
- The widget itself is purely a display layer; it receives `ActivityViewContext<WorkoutWidgetAttributes>` and renders `context.state.metrics` and `context.attributes`.
- Distance display unit respects `Defaults[.unitSystem]` read at render time.
- Pace shown as `metrics.paceDisplay` formatted string (computed in `MetricsModel`).
- Heart rate displayed as `"%.0f"` formatted double or `"--"` if nil/zero.
- `isFinished` flag triggers the green checkmark overlay.

## Premium
Requires iOS 26+. No Pro subscription gating, but the Live Activity only appears during an active workout session tracked by the app.

## Related Files
- `/Users/hieudinh/Projects/Steps/StepsWidget/LiveWorkoutWidget/WorkoutWidgetLiveActivity.swift`
- `/Users/hieudinh/Projects/Steps/StepsWidget/LiveWorkoutWidget/WorkoutWidgetAttributes.swift`
- `/Users/hieudinh/Projects/Steps/StepsWidget/LiveWorkoutWidget/LiveActivityView.swift`

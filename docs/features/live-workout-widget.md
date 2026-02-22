# Live Workout Widget

## Overview
A Live Activity (Dynamic Island / Lock Screen widget) that surfaces real-time workout metrics while a session is active. Starts when the workout begins, updates every ~2 seconds via the timer loop, and dismisses automatically 60 seconds after the workout ends.

## User Experience
1. When the user starts a workout, a Live Activity appears on the Lock Screen and in the Dynamic Island.
2. The widget shows the workout type SF Symbol, workout name, elapsed time, distance, active energy, and pace/speed — mirroring the in-app session metrics.
3. The widget updates every ~2 seconds (the `updateWorkoutLiveActivity` function has a 2-second sleep before each push).
4. A stale date of `now + 15` seconds is set on each update, so the system marks the widget stale if no update arrives within 15 seconds.
5. When the workout ends, the widget transitions to a "finished" state (via `isFinished: true` in `ContentState`) and is automatically dismissed 60 seconds later using `.after(.now + 60)` dismissal policy.
6. If Live Activities are not enabled on the device (`ActivityAuthorizationInfo().areActivitiesEnabled == false`), the feature silently skips activation and sets `WorkoutManager.isLiveActivityActive = false`.
7. If the app is killed mid-workout and the session is recovered, `startLiveActivity` is called again during `recoverWorkout`, resuming the Live Activity.

## Key Components
- `WorkoutWidgetViewModel` - `@Observable` `@MainActor` singleton (`shared`) managing the full Live Activity lifecycle.
- `ActivityViewState` - inner struct tracking `activityState` (`.active`, `.stale`, `.ended`, `.dismissed`, `.pending`), `contentState`, push token, and derived booleans `shouldShowEndControls`, `shouldShowUpdateControls`, `isStale`, `updateControlDisabled`.
- `WorkoutWidgetAttributes` - `ActivityAttributes` conforming type (defined in the widget target) carrying static data: `symbol` (SF Symbol name), `workoutActivityType` (raw `UInt`), `workoutName`.
- `WorkoutWidgetAttributes.ContentState` - dynamic state carrying a `MetricsModel` snapshot and an `isFinished` flag.
- `startLiveActivity(symbol:workoutActivityType:workoutName:)` - creates the `Activity<WorkoutWidgetAttributes>` with `.token` push type and calls `setup(withActivity:)`.
- `updateLiveActivity(metrics:)` - schedules an async task that sleeps 2 seconds then calls `updateWorkoutLiveActivity(metrics:)` to push a new `ActivityContent` with `staleDate: now + 15` and `relevanceScore: 100`.
- `endLiveActivity(dismissTimeInterval:metrics:)` - sends a final `ContentState` with `isFinished: true` and a `.after(now + 60)` dismissal policy.
- `observeActivity(activity:)` - spawns two concurrent tasks in a `TaskGroup`: one watches `activityStateUpdates` (cleans up on `.dismissed`), the other watches `contentUpdates` to keep `activityViewState.contentState` in sync.
- `cleanUpDismissedActivity()` - nils out `currentActivity` and `activityViewState`.
- `WorkoutManager.startWorkoutTimer()` - fires every 1 second, updates `metrics.elapsedTime` from `builder?.elapsedTime`, then calls `WorkoutWidgetViewModel.shared.updateLiveActivity(metrics:)`.

## Data & Logic
- The Live Activity is requested with `.token` push type, enabling server-side push updates if desired (push token is stored in `activityViewState.pushToken` as a hex string).
- Update cadence: the 1-second app timer triggers `updateLiveActivity` on every tick, but `updateWorkoutLiveActivity` introduces a 2-second sleep, effectively throttling pushes to ~0.5 Hz.
- Metrics pushed: full `MetricsModel` snapshot — elapsed time, heart rate, active energy, distance, speed, pace (computed).
- State machine integration: `WorkoutManager` calls `startLiveActivity` in both `startWorkout()` and `recoverWorkout(recoveredSession:)` to handle app-restart recovery.
- `isLiveActivityActive` flag on `WorkoutManager` is set to `true` on successful start and `false` on end or failure.

## Premium
No premium gating. The Live Activity widget is available to all users.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Workout/WorkoutWidgetViewModel.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Workout/WorkoutManager.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Workout/MetricsModel.swift`

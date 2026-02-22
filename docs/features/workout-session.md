# Workout Session

## Overview
The active workout screen shown during a live workout. Displays a real-time map with the route being traced, a live stats grid, and pause/resume/end controls. Requires iOS 26+.

## User Experience
1. After the 3-second countdown completes, `SessionView` replaces `CountDownView` inside `WorkoutHomeView`.
2. The top portion of the screen shows a live map centered on the user's current location (`MapCameraPosition.userLocation`). The route polyline traces the path taken so far in a gradient from 90% to 40% opacity of the workout color. A start-point annotation uses the workout type's SF Symbol inside a colored glass circle.
3. `UserAnnotation()` shows the standard blue dot for the user's current position. A "recenter" button (`MapUserLocationButton`) and compass are available as map controls.
4. The bottom card shows a 2-column stats grid:
   - **Duration** - live elapsed time via `ElapsedTimeView` + `MetricsTimelineSchedule` (updates at 30 fps when running, paused when session is paused).
   - **Distance** - cumulative distance in the user's preferred unit, sourced from HealthKit live statistics.
   - **Active Energy** - kilocalories burned, from HealthKit.
   - **Fourth stat** - varies by activity type:
     - Cycling: **Speed** (km/h or mph), throttled every 5 seconds.
     - Hiking: **Heart Rate** (bpm), live from HealthKit.
     - Running/Walking: **Pace** (min/km or min/mi), throttled every 5 seconds.
5. Current weather (temperature + humidity) is displayed in the navigation bar center, fetched via WeatherKit at workout start and refreshed every 5 minutes.
6. The navigation bar shows:
   - **Left** (only when paused): "End" button (stop icon) — ends and saves the workout.
   - **Center**: workout name (e.g., "Outdoor Running") + weather.
   - **Right**: "Pause" button (pause icon) when running; "Resume" button (clockwise arrow) when paused.
7. Tapping Pause: session pauses, route collection stops, timer stops. The End button appears on the left.
8. Tapping Resume: session resumes, route collection and timer restart.
9. Tapping End: `workoutManager.endWorkout()` is called. A glass overlay with "Saving workout" spinner covers the screen while HealthKit finalizes the workout and route. After saving, the app navigates directly to the new workout's detail view.
10. If the workout session was interrupted (e.g., app killed), it can be recovered via `recoverWorkout(recoveredSession:)` which restores persisted route locations from `UserDefaults` and restarts the Live Activity.

## Key Components
- `SessionView` - main view; owns throttled pace/speed state and weather timer.
- `WorkoutHomeView` - navigation container; uses `WorkoutNavigationModel` to switch between `CountDownView` and `SessionView` based on `WorkoutManager.state`.
- `WorkoutNavigationModel` - `@Observable` class that observes `WorkoutManager.state` via `withObservationTracking` and maps it to `.startView`, `.countdownView`, or `.sessionView`.
- `ElapsedTimeView` - displays elapsed time formatted as MM:SS using `ElapsedTimeFormatter` (`DateComponentsFormatter` with `.minute` and `.second` units, zero-padded).
- `MetricsTimelineSchedule` - custom `TimelineSchedule` that drives `TimelineView` at 30 fps when running, and returns `nil` (no updates) when paused.
- `WorkoutManager` - singleton `@Observable` class managing the HealthKit session, route builder, and metrics. See workout-session.md for full details.
- `MetricsModel` - value type holding all live metrics: `elapsedTime`, `heartRate`, `activeEnergy`, `distance`, `speed`, `pace` (computed from speed or distance/time fallback).
- Pace/Speed throttle: `Timer.publish(every: 5.0)` updates `throttledPace` and `throttledSpeed` only while the session is `.running`, preventing jitter from instant HealthKit updates.
- Weather timer: `Timer.publish(every: 300.0)` re-fetches weather via `LocationManager.fetchWeatherForWorkout()`.

## Data & Logic
- `WorkoutManager` uses `HKWorkoutSession` + `HKLiveWorkoutBuilder` + `HKLiveWorkoutDataSource` for all HealthKit metric collection.
- Live statistics arrive via `workoutBuilder(_:didCollectDataOf:)` delegate and are dispatched to `updateForStatistics(_:)`, which fans out to the correct `MetricsModel` field by quantity type.
- Distance quantity type is activity-dependent: `distanceWalkingRunning` for walk/run/hike, `distanceCycling` for cycling.
- Speed quantity type: `cyclingSpeed`, `runningSpeed`, or `walkingSpeed`. Uses `mostRecentQuantity()` (not sum) so it reflects current speed, not average.
- Pace = `1000 / (60 * speed_m_s)` when HealthKit speed is available; falls back to `distance_km / elapsed_minutes` otherwise.
- Route polyline: read from `Defaults[.workoutSessionLocations]` (persisted `WorkoutSessionLocation` structs) and converted back to `CLLocationCoordinate2D` for `MKPolyline`.
- Speed display for cycling: tries HealthKit speed first; falls back to `distance / elapsedTime`.
- After `endWorkout()`, `consumeSessionStateChange` finalizes the builder, ends the Live Activity (dismiss after 60 s), saves weather data from `Defaults[.currentWeatherData]`, calls `DataStore.shared.upsertWorkout(from:temperature:humidity:)`, then navigates to the new workout.

## Premium
No features in the active workout session are Pro-gated. All real-time metrics, the map, and route tracing are available to all users.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Workout/SessionView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Workout/WorkoutHomeView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Workout/WorkoutManager.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Workout/MetricsModel.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Workout/ElapsedTimeView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Workout/CountDownView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Workout/CountDownManager.swift`

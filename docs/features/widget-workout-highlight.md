# Widget: Workout Highlight

## Overview
A small home screen widget that cycles through five different workout highlight categories, surfacing one standout workout per hour. Tapping the widget deep-links directly to that workout's detail screen in the app.

## User Experience
- Displayed as a `.systemSmall` widget.
- Header badge (orange-gradient pill) shows the highlight category name: "Recent Workout", "Longest Workout", "Longest Distance", "Fastest Pace", or "Most Calories".
- Below the badge: workout icon in a colored circle (color varies by workout type), workout name (e.g., "Outdoor Run"), and a relative timestamp ("Today" if today, otherwise the date).
- A 2×2 compact metric grid shows Distance, Duration, Calories, and Avg Pace (pace omitted for Cycling and Hiking).
- The accent color for the icon circle and background gradient matches the workout type: Running → purple, Cycling → green, Walking → blue, Hiking → brown, others → app accent.
- If no workouts exist, shows an empty-state view with a flame icon, "No workouts yet" heading, and "Start a workout to see highlights here" message.
- Tapping a workout card opens `steps://open?screen=WORKOUT_DETAIL&workoutId=<uuid>` deep link.
- The timeline generates one entry per highlight category (shuffled order), each 1 hour apart. After all 5 entries expire the timeline refreshes.

## Key Components
- **`WorkoutHighlightsWidget`** — Static configuration, `.systemSmall` only, kind `"WorkoutHighlightsWidget"`.
- **`WorkoutHighlightsProvider`** — Timeline provider. Builds 5 entries (one per `WorkoutHighlight` case, shuffled), each offset by 1 hour from now.
- **`WorkoutHighlight` enum** — `.recent`, `.longestDuration`, `.longestDistance`, `.fastestPace`, `.mostCalories`. Each has a `title` and empty-state strings.
- **`WorkoutHighlightsDisplayData`** — Pre-formatted display struct: `highlightTitle`, `iconName`, `workoutName`, `dateText`, `timeText`, `distanceText`, `distanceUnit`, `caloriesText`, `durationText`, `paceText?`, `accentColor`, `hasWorkout`.
- **`WorkoutHighlightsWidgetView`** — Renders `smallWorkoutView` (has workout) or `smallEmptyView` (no workout). Uses `widgetURL` modifier for deep-link tap.
- **`CompactMetricView`** — Small label + value column used in the 2×2 metric grid.
- **Highlight selection logic** — `.recent` → max `startDate`; `.longestDuration` → max `duration`; `.longestDistance` → max `distance`; `.fastestPace` → min `averagePace` (excludes Cycling and Hiking); `.mostCalories` → max of `activeEnergy ?? totalEnergy`.
- **Indoor/outdoor prefix** — If `workout.isIndoor` is set, prepends "Indoor " or "Outdoor " to the activity display name.
- **Pace formatting** — `DateComponentsFormatter` with `.abbreviated` style, max 2 units, drops leading zeros.

## Data & Logic
- Data source: `Defaults[.workouts]` — full `[WorkoutData]` array in the shared App Group container.
- Filters to `workout.isDisplayedInActivity == true` before applying highlight logic.
- Distance stored in meters, divided by 1000 for km, then converted to user unit system.
- Calories: prefers `activeEnergy`, falls back to `totalEnergy`.
- Duration formatted via `DateComponentsFormatter` with hour/minute/second, max 2 units, `.abbreviated` style.
- Pace converted via `unitSystem.convertedPace(fromMinutesPerKilometer:)`.

## Premium
No explicit gating in widget code. Workout data itself may depend on HealthKit access and whether the user has performed workouts tracked by the app.

## Related Files
- `/Users/hieudinh/Projects/Steps/StepsWidget/WorkoutHighlightWidget/WorkoutHighlightWidget.swift`
- `/Users/hieudinh/Projects/Steps/StepsWidget/WorkoutHighlightWidget/WorkoutHighlightsWidgetView.swift`

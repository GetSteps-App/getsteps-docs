# Workout List

## Overview
A scrollable list of all startable workout types presented as selectable cards. Serves as the entry point for choosing which workout to begin. Requires iOS 26+.

## User Experience
1. The user navigates to the Workout tab. `WorkoutListView` is displayed inside a `NavigationStack` titled "Workouts".
2. Each supported workout type appears as a card row with:
   - The workout type's SF Symbol icon in its designated color (left side).
   - The workout name in semibold text (e.g., "Outdoor Running", "Indoor Walking").
   - An orange play circle button (`play.circle.fill`, 48×48 pt with a glass effect) on the right.
3. Tapping the play button triggers a warning haptic (`UINotificationFeedbackGenerator`) and sets `workoutManager.selectedWorkout` to that configuration.
4. Setting `selectedWorkout` immediately calls `prepareWorkout()` on `WorkoutManager`, which creates the `HKWorkoutSession` and transitions the state to `.prepared`, which in turn triggers `WorkoutNavigationModel` to show `CountDownView`.
5. HealthKit authorization is requested once on appear (skipped in Xcode previews).
6. The list also appears implicitly from the Activity tab's "+" toolbar menu on iOS 26+, which sets `selectedWorkout` directly and switches to the Workout tab — bypassing this list view.

## Key Components
- `WorkoutListView` - the sole view; reads `WorkoutTypes.workoutConfigurations` to build the list.
- Each row uses `cardStyle` modifier for consistent card appearance.
- `WorkoutManager.selectedWorkout` setter is the activation trigger; setting it kicks off `prepareWorkout()` asynchronously.

## Data & Logic
- The card list is generated from `WorkoutTypes.workoutConfigurations`, a computed property that:
  - Iterates over `WorkoutTypes.startable` (walking, running, cycling, hiking).
  - For types where `shouldDisambiguateLocation` is true (walking, running, cycling), produces both an Outdoor and an Indoor `HKWorkoutConfiguration`.
  - For hiking (no indoor/outdoor split), produces one configuration.
  - Result: 7 configurations total (Outdoor Walk, Indoor Walk, Outdoor Run, Indoor Run, Outdoor Cycle, Indoor Cycle, Hike).
- `requestAuthorization()` requests write access to `HKQuantityType.workoutType()` and `HKSeriesType.workoutRoute()`, and read access to energy, heart rate, distance, speed, and route types.

## Premium
No part of the workout list or workout initiation is Pro-gated.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Workout/WorkoutListView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Workout/WorkoutTypes.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Workout/WorkoutManager.swift`

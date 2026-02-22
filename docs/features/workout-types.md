# Workout Types

## Overview
A static configuration layer that defines which HealthKit workout activity types the app supports, how they are categorized, what quantity types they use for distance/speed, and how they are visually represented (icon, color, name).

## User Experience
Users do not interact with this feature directly. It drives the icons, colors, and names shown throughout the app — in the workout list, activity list rows, detail view headers, share images, and Live Activity widget.

## Key Components
- `WorkoutTypes` struct - namespace for all type-related static data and logic.
- `WorkoutTypes.startable` - the 4 types users can initiate from the app: `.walking`, `.running`, `.cycling`, `.hiking`.
- `WorkoutTypes.tracked` - 23 named HealthKit types read from HealthKit history (includes swimming, HIIT, strength, yoga, pilates, elliptical, rowing, dance, racket sports, etc.).
- `WorkoutTypes.displayedInActivity` - the 4 types shown in the Activity List view (walk, run, cycle, hike).
- `WorkoutTypes.isOtherType(_:)` - returns `true` for any type not in `tracked`; used to bucket unknown types as "Other".
- `WorkoutTypes.shouldDisambiguateLocation(for:)` - returns `true` for walking, running, cycling, rowing, swimming — these produce both Indoor and Outdoor configurations.
- `WorkoutTypes.workoutConfigurations` - computed array of `HKWorkoutConfiguration` objects; produces Indoor+Outdoor pairs for disambiguated types, single configs for others.
- `WorkoutTypes.distanceQuantityType(for:)` - maps activity type to the correct `HKQuantityType`: `distanceWalkingRunning` for walk/run/hike, `distanceCycling` for cycling, `nil` for others.
- `WorkoutTypes.speedQuantityType(for:)` - maps to `cyclingSpeed`, `runningSpeed`, or `walkingSpeed`; `nil` for types without a speed metric.

### Extensions

`HKWorkoutConfiguration` extensions:
- `name` - combines location type ("Outdoor"/"Indoor") with activity name when disambiguation applies (e.g., "Outdoor Running"); plain activity name otherwise.
- `symbol` - SF Symbol name derived from `workoutTypeString` + `workoutSymbolName` helpers; indoor variants use a different symbol than outdoor.
- `color` - per-type `Color`: running = `.purple`, walking = `.blue`, cycling = `.green`, hiking = `.mint`, default = `.urange` (app accent).

`HKWorkoutActivityType` extensions:
- `name` - localized display name via `workoutTypeString` + `workoutName` helpers.
- `color` - full color map for all 23 tracked types plus a `.gray` default for unmapped types.

`HKWorkoutSessionLocationType` extension:
- Retroactive `CustomStringConvertible` conformance: `.indoor` → "Indoor", `.outdoor` → "Outdoor", `.unknown` → "Unknown".

## Data & Logic
- All configuration is static/computed — no network calls or persistence.
- `workoutTypeString` and `workoutName`/`workoutSymbolName` are global helper functions (defined elsewhere in the codebase) that provide the string-based type identifier and localized display strings used by both extensions.
- The indoor symbol differs from the outdoor symbol for the same activity (e.g., treadmill icon vs. running figure).
- Color assignments are hardcoded per type to ensure consistent brand identity across all views and exported share images.

## Premium
No premium gating. All type metadata is available to all users.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Workout/WorkoutTypes.swift`

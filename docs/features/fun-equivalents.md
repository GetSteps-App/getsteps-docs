# Fun Equivalents

## Overview

Fun Equivalents translates raw yearly step and workout numbers into relatable, narrative comparisons. It appears twice in the Insights screen: once for daily walking stats (distance, calories, CO2 saved, body fat burned) and once for workout-specific stats (workout duration and workout distance). The text cycles through multiple creative phrasings to keep the copy fresh across different activity levels.

## User Experience

1. After the metric cards section, a block of styled narrative text appears on a transparent background with large faded emoji watermarks in each corner.
2. Two sentences are shown stacked:
   - A "fun message" combining the distance equivalent and calories equivalent (e.g. "This year, you walked **the distance from New York to LA**, and burned **2,340 slices of pizza**!").
   - An "impact message" combining the CO2 saved equivalent and body fat burned (e.g. "You also saved **12 trees worth of CO2**, and burned **1.2 kg of body fat**").
3. The intro phrase for each sentence rotates through 6 variants based on `totalSteps % 6` so the phrasing differs between users and years.
4. Workout Fun Equivalents (shown lower in Insights, only with workouts) uses 4 variants each for duration and distance messages, rotating on `totalWorkouts % 4`.
5. Key values (distance, calories, CO2, body fat, duration equivalent, distance equivalent) are rendered in a larger bold font within the sentence via SwiftUI `Text` concatenation.

## Key Components

- `FunEquivalentsView` — walking stats equivalents block.
  - Four background emoji watermarks at 15% opacity (distance emoji top-left, calories bottom-right, CO2 bottom-left, body fat bottom-right).
  - `funMessageText` — 6 intro variants wrapping distance + calories values.
  - `impactMessageText` — 6 intro variants wrapping CO2 + body fat values.
- `WorkoutFunEquivalentsView` — workout stats equivalents block.
  - Two background emoji watermarks (duration emoji top-left, distance emoji bottom-right).
  - `durationMessageText` — 4 variants comparing total workout time to a pop-culture duration (e.g. number of movies).
  - `distanceMessageText` — 4 variants contextualising total workout distance.

## Data & Logic

- Data comes from `DataStore` computed properties:
  - `yearlyDistanceEquivalent`, `yearlyDistanceDescription`, `yearlyDistanceEmoji`
  - `yearlyCaloriesEquivalent`, `yearlyCaloriesDescription`, `yearlyCaloriesEmoji`
  - `yearlyCO2Value`, `yearlyCO2Description`, `yearlyCO2Emoji`
  - `yearlyBodyFatValue`, `yearlyBodyFatDescription`, `yearlyBodyFatEmoji`
  - `yearlyWorkoutDurationEquivalent`, `yearlyWorkoutDistanceDescription`, `yearlyWorkoutDurationEmoji`
  - `yearlyWorkoutDistanceEquivalent`, `yearlyWorkoutDistanceEmoji`
- The `messageIndex` parameter is `yearlyTotalSteps` (for walking) or `yearlyTotalWorkouts` (for workouts); modulo arithmetic selects the phrase variant.
- If `distanceDescription` is empty (no data), only the bold value is shown without a description suffix.
- No network calls; all computation is local in `DataStore`.

## Premium

This feature is **free**. No Pro subscription required.

## Related Files

- `/Users/hieudinh/Projects/Steps/Steps/Insights/FunEquivalentsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/WorkoutFunEquivalentsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/InsightsView.swift`

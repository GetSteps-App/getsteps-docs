# Personal Records

## Overview

Personal Records is a section within the Insights screen that surfaces the user's single best workout achievement across 8 distinct categories for the selected year. Tapping a record card highlights the corresponding workout on a live MapKit map directly below the card grid.

## User Experience

1. The section appears in Insights after the Workout Breakdown section, visible only when `yearlyTotalWorkouts > 0`.
2. A horizontal scrollable grid (2 rows, peek of next column visible) displays 8 record cards.
3. Each card shows an icon, category title, record value, and the date it occurred.
4. The selected card has a coloured border and slight scale-up (1.05x). Tapping another card animates the selection change.
5. Below the grid, a `HighlightedWorkoutView` map updates to show the route of the selected record's workout.
6. The map shows a polyline in the record's accent colour, a start annotation (workout type icon), and a finish annotation (checkered flag). If no GPS route exists, a "No route available" placeholder appears.
7. The map header shows a contextual sentence ("Your fastest pace was X:XX /km") that changes with selection via a content transition.
8. The map footer shows the workout's type, indoor/outdoor label, date, duration, distance, calories, and weather (temperature + humidity if available).

## Key Components

- `PersonalRecordsView` — container with a section header ("Your personal records 🏆") and a horizontal `LazyHGrid`.
- `PersonalRecordCard` — individual tappable card; shows icon, title, value, optional subtitle (date), accent colour highlight, and selected state border.
- `PersonalRecordType` enum — defines 8 cases with title, SF Symbol icon, and accent colour:
  - `fastestPace` (purple, bolt.fill)
  - `mostCalories` (orange, flame.fill)
  - `earliest` (yellow, sunrise.fill)
  - `mostElevation` (green, mountain.2.fill)
  - `longest` (blue, timer)
  - `farthest` (teal, point.topleft.down...)
  - `latest` (indigo, moon.fill)
  - `maxHeartRate` (red, heart.fill)
- `HighlightedWorkoutView` — MapKit map with variable-blur overlays at top and bottom; non-interactive (`allowsHitTesting(false)`); camera auto-fits route bounds with 2x lat / 1.5x long padding.

## Data & Logic

- Each record type maps to a `WorkoutData?` property on `DataStore` (e.g. `yearlyFastestPaceWorkout`, `yearlyMostCaloriesWorkout`).
- `DataStore.fetchRecentWorkouts()` populates these on Insights appear.
- Value formatting per type:
  - Pace: stored as min/km (Double), converted to min/mi for imperial via ×1.60934.
  - Calories: formatted with no decimal, suffixed "kcal".
  - Time (earliest/latest): `h:mm a` format from `startDate`.
  - Elevation: meters or feet (imperial ×3.28084).
  - Duration: hours + minutes.
  - Distance: km or miles from `workout.distance / 1000`.
  - Heart rate: integer bpm.
- Subtitle for all records is the workout date formatted as "MMM d".
- If no data exists for a category, the value shows "--".
- Route fetching: `dataStore.fetchRouteForWorkout(_:completion:)` returns `[CLLocation]`; polyline is built from coordinates; camera region is computed from lat/long min/max.
- Weather display: temperature shown with colour-coded thermometer symbol; humidity shown with colour-coded humidity symbol.

## Premium

Personal Records is **free**. However, the route map within `HighlightedWorkoutView` uses the same map infrastructure as the Pro Route Map feature; the records map itself has no paywall gate.

## Related Files

- `/Users/hieudinh/Projects/Steps/Steps/Insights/PersonalRecordsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/HighlightedWorkoutView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/InsightsView.swift`

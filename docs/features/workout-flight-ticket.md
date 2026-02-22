# Workout Flight Ticket

## Overview

The Workout Flight Ticket is a boarding-pass styled card in the Insights screen that summarises the user's yearly workout totals. It uses airline ticket visual language — a "JAN → DEC" route, a perforated tear line separating the main ticket from the stub, and a QR code linking to the app's website. It only appears when the user has at least one workout in the selected year.

## User Experience

1. Appears in the Workouts section of Insights, directly after the "Workouts" section header.
2. The ticket header shows "BOARDING PASS" in caption2 and "[YEAR] FITNESS" in headline, with a blue gradient `airplane.path.dotted` icon on the right.
3. The route section shows "JAN" on the left (START) and "DEC" on the right (FINISH), with a flight-path graphic in between: dot — line — dumbbell icon — line — dot, with the total workout count labelled below in orange.
4. A divider separates the route from a stats grid of three columns: DURATION, DISTANCE, BURNED (calories). Each value is bold, with auto-scaling to fit.
5. A perforated divider (larger notch radius than the receipt version) separates the main ticket from the stub.
6. The stub shows a rotating "boarding message" on the left (e.g. "READY FOR TAKEOFF 🛫", "FIRST CLASS FITNESS ✨", "CLEARED FOR GAINS 💪") and on the right: the app icon (24×24, rounded), the "GetSteps.app" label, and a 40×40 QR code.
7. The QR code is generated asynchronously on a `.utility` priority detached task and regenerates when the colour scheme changes.
8. The whole card has rounded corners (16 pt), a drop shadow, and background colour matching the receipt (system background / 5% white).

## Key Components

- `WorkoutFlightTicketView` — the full boarding-pass card.
- `TicketPerforatedDivider` — shared shape from `PerforatedEdgeShapes.swift`; notch radius 8 for the ticket stub tear.
- `DashedLine` — dashed stroke used for the stub divider line.
- QR code: generated via the `QRCode` library at 120px (40pt × 3x scale). Uses `Square` pixel and eye shapes. Foreground/background colours adapt to colour scheme.

## Data & Logic

- Inputs passed from `InsightsView`:
  - `totalWorkouts` — `"\(dataStore.yearlyTotalWorkouts)"`
  - `workoutTime` — formatted as "Xh Ym" from `yearlyTotalWorkoutDuration` (seconds)
  - `workoutDistance` — converted distance from `yearlyTotalWorkoutDistance / 1000` km
  - `workoutCalories` — `yearlyTotalWorkoutCalories` formatted with 0 decimal places
  - `messageIndex` — `yearlyTotalWorkouts`; modulo 8 selects the boarding message
- `currentYear` reads `dataStore.selectedInsightYear`.
- QR code URL: `"https://getsteps.app"` (hardcoded).
- QR generation: `Task.detached(priority: .utility)` → result returned to `MainActor`.
- `onChange(of: colorScheme)` triggers QR regeneration with updated foreground/background colours.

## Premium

This feature is **free**. No Pro subscription required.

## Related Files

- `/Users/hieudinh/Projects/Steps/Steps/Insights/WorkoutFlightTicketView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/PerforatedEdgeShapes.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/InsightsView.swift`

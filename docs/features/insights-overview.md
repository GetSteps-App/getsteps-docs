# Insights Overview

## Overview

The Insights screen is a yearly recap of the user's step and workout activity. It aggregates data from HealthKit for a selected year (2025 or 2026) and presents it as a scrollable summary using themed, receipt- and ticket-style UI components. The tab is introduced to first-time viewers via an intro sheet (`InsightsIntroSheetView`).

## User Experience

1. User navigates to the Insights tab (tab index 2).
2. On first open, `hasShownInsightsIntroSheet` is set to true (intro sheet shown elsewhere in the app before landing here).
3. A year picker (2025 / 2026) appears in the top-left toolbar menu; changing the year reloads all data with animation.
4. The screen scrolls through a sequence of sections:
   - **Steps section header** — bold secondary label.
   - **Yearly Receipt** — styled receipt card showing total steps, distance, and calories.
   - **Daily Average + Best Day** — two `MovieTicketMetricCard` tiles side by side.
   - **Weekday Steps Breakdown** — bar chart + weekday vs weekend comparison bar.
   - **Best Month, Peak Hour, Goal Met, Best Streak** — four `RecapMetricCard` / `NewRecapMetricCard` tiles in a 2x2 grid.
   - **Fun Equivalents** — narrative sentences translating distance, calories, CO2, and body fat into relatable comparisons.
   - **Workouts section** (only shown if `yearlyTotalWorkouts > 0`):
     - **Workout Flight Ticket** — boarding-pass card summarising total workouts, time, distance, calories.
     - **Workout Breakdown** — pie chart + indoor/outdoor/weekday/weekend split.
     - **Personal Records** — horizontal scrollable grid of 8 PR cards; tapping one highlights the corresponding workout on a map below.
     - **Workout Fun Equivalents** — narrative sentences for workout duration and distance.
   - **Passport Stamp** — decorative stamp showing the year and "COMPLETE" or "AWAITS".
   - **Motivational Sign-Off** — contextual message based on total steps / workouts.
5. On appear, metric values animate in with a 0.25 s delay; on disappear they reset (used to re-trigger animation on next visit).
6. On iOS 26+, a gear icon in the top-right opens `SettingsView`.

## Key Components

- `InsightsView` — root container; owns year picker, data routing, and scroll layout.
- `YearlyReceiptView` — receipt-styled card with perforated edges showing steps, distance, calories, and a rotating "total" message.
- `MovieTicketMetricCard` — card for Daily Average and Best Day.
- `WeekdayStepsBreakdownView` — animated bar chart (bars stagger in sequentially) + split comparison bar for weekday vs weekend averages.
- `RecapMetricCard` / `NewRecapMetricCard` — small metric tiles for Best Month, Peak Hour, Goal Met, Best Streak.
- `FunEquivalentsView` — two-paragraph text block with background emoji watermarks; cycles through 6 intro phrases and 6 impact phrases based on total steps.
- `WorkoutFlightTicketView` — boarding-pass card with QR code linking to `https://getsteps.app`; QR is generated asynchronously via the `QRCode` library, regenerates on color-scheme change.
- `WorkoutBreakdownView` — donut/pie chart using Swift Charts, plus outdoor/indoor and weekday/weekend counts.
- `PersonalRecordsView` — horizontal `LazyHGrid` (2 rows × 4 columns) of tappable record cards; selected card drives the map below.
- `HighlightedWorkoutView` — square MapKit map overlay showing route polyline + start/finish annotations; fetches GPS route from `DataStore`; shows loading spinner or "No route available" placeholder.
- `WorkoutFunEquivalentsView` — narrative sentences for workout duration and distance equivalents.
- `PassportStampView` — rotated (-12°) notched-border stamp; shows "AWAITS" for current year, "COMPLETE" for past years.
- `MotivationalSignOffView` — sign-off message and year look-ahead; message tier based on total steps (5k/7k/10k per day averages) and workout count.
- `InsightsIntroSheetView` — medium-detent intro sheet with animated auto-scrolling emoji row; auto-advances scroll via a 0.01s timer.

## Data & Logic

- All yearly metrics are computed properties on `DataStore` (e.g. `yearlyTotalSteps`, `yearlyTotalDistance`, `yearlyBestDay`, `yearlyPeakHour`, `yearlyWeekdayAverageSteps`, `yearlyTotalWorkouts`, `yearlyFastestPaceWorkout`, etc.).
- The `selectedInsightYear` on `DataStore` (2025 or 2026) drives which slice of data is shown.
- `refreshWorkoutsAsync()` calls `dataStore.fetchRecentWorkouts()` on appear.
- Weekday breakdown: step records are grouped by `Calendar.weekday`; order respects the user's `weekStartsOn` (Sunday or Monday) preference.
- Best day, peak hour, and best month are derived server-side / in `DataStore` from the yearly step records.
- Formatters (number, distance, calories, date) are cached as static properties for performance.
- Unit system (`UnitSystem` default) controls distance labels (km vs mi) and temperature.
- Route fetching for `HighlightedWorkoutView` dispatches to `dataStore.fetchRouteForWorkout(_:completion:)` on main thread.

## Premium

This feature is **free** — no Pro subscription required to view the Insights screen or any of its content.

## Related Files

- `/Users/hieudinh/Projects/Steps/Steps/Insights/InsightsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/InsightsIntroSheetView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/YearlyReceiptView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/WeekdayStepsBreakdownView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/RecapMetricCard.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/FunEquivalentsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/WorkoutFlightTicketView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/WorkoutBreakdownView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/PersonalRecordsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/HighlightedWorkoutView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/WorkoutFunEquivalentsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/PassportStampView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/MotivationalSignOffView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/PerforatedEdgeShapes.swift`

# Step Count Card

## Overview
The step count card is the primary data display component on the home screen. It shows the current step count, distance, calories, and flights climbed for the selected date or period. The card supports swipe gestures to navigate between days, weeks, or months, and renders a progress shadow ring tied to the daily step goal.

## User Experience
1. The card occupies the upper portion of the home screen, displaying a large serif step count number, secondary metrics below it, and an optional insight area at the bottom.
2. The date heading appears as weekday name (daily) / "Week N" (weekly) / year (monthly). Below it a sub-heading shows the full date, date range, or month name.
3. Tapping the heading area opens a popover picker to change the calendar view mode (Daily / Weekly / Monthly).
4. Swiping the card left or right navigates to the next or previous period. Navigation to future dates is blocked. The animation shows the incoming card sliding in from the appropriate side while the outgoing card slides away; a "stacked" ghost card peeks behind during the gesture.
5. A flame badge in the top-left shows the current streak count in orange (grey when 0). Tapping it opens `StreaksSheetViewDetailed`.
6. A refresh button in the top-right re-fetches step data from HealthKit with a spinning arrow animation. After completion a "Step count updated" toast appears.
7. In weekly/monthly mode a toggle button (sum/average icon) switches between total and daily-average display. The watch is updated via `WatchConnectivityManager` when this changes.
8. Tapping anywhere on the card body triggers a ripple effect animation from the tap point.
9. When the step goal is first reached, a completion ripple fires from the card center with haptic feedback, triggered via `dataStore.onProgressCompleted`.
10. At the bottom of the card an optional insight is shown: either a motivational message or an `AverageStepComparisonView` depending on `homeInsightMode`.

## Key Components
- `StepCountCardView` — gesture container that manages `dragOffset`, `scale`, and `frontCardOpacity` state. Composites `StepCountContentView` (front, interactive) and two `StepCountContentStackedView` instances (back "ghost" cards for the swipe peek effect).
- `StepCountContentView` — the main content layer. Reads from `DataStore`, computes `displayStepCount`, `displayDistance`, `displayCalories`, `displayFlights`, `goalProgress`, and `displayMessage`. Contains all interactive overlays (streak badge, refresh button, mode toggle, sheets).
- `StepCountContentStackedView` — a non-interactive visual copy of the card used as the peek card during swipe; updates its own display values via `onChange(of: selectedDate)`.
- `AverageStepComparisonView` — renders a 20-bar ASCII-style progress indicator and difference text comparing current steps vs historical average.
- `StreaksSheetViewDetailed` — bottom sheet opened by the flame badge (documented in streaks.md).
- `RippleEffect` modifier — custom view modifier creating a ripple animation from a given point.
- `progressShadow` modifier — draws a rounded-rect progress ring shadow in `goalInProgress` or `goalCompleted` color with 6pt line width and 10pt blur.

## Data & Logic
- `displayStepCount`:
  - Daily mode: `dataStore.stepCount` for today; `stepDailyRecords` lookup for a chart-selected past date.
  - Weekly/monthly mode with average toggle: arithmetic mean of all non-future `StepDailyRecord` entries in the period.
  - Weekly/monthly mode with total toggle: `dataStore.stepCount` (which the DataStore sets to the period total when those modes are active).
- `goalProgress`:
  - Daily: `displayStepCount / historicalGoalForDate` (from `goalChangeLog`).
  - Weekly: `displayStepCount / (stepGoal * 7)`.
  - Monthly: `displayStepCount / (stepGoal * daysInMonth)`.
- `heading` / `subHeading` change based on `calendarViewMode` and whether `chartDateSelection` is set (a tapped bar in the chart overrides the heading to show that specific day).
- Swipe navigation increments/decrements by 1 day, 1 week (to week start), or 1 month (to month start), depending on `calendarViewMode`. A minimum swipe of 50pt is required to commit the navigation.
- `WatchConnectivityManager.sendStepGoalToWatch(...)` is called when `stepCountMode` is toggled to keep the Apple Watch widget in sync.

## Premium
- The sum/average toggle button (`stepCountMode`) is only visible when `calendarViewMode.canShowAverage` is true, which requires Pro (weekly or monthly mode). The flame streak badge and goal ring are available to all users when `enableDailyStepGoal` is on.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Home/StepCountCardView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Home/StepCountContentView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Home/StepCountContentStackedView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Home/AverageStepComparisonView.swift`

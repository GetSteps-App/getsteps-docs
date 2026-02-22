# Widget: Yearly Step Calendar

## Overview
A large home screen widget showing an 18×21 grid of tiny cells — one per day of the year — color-coded by activity level. Inspired by GitHub's contribution graph. Also surfaces the current streak, total days goal reached, and average daily steps for the year.

## User Experience
- Displayed as a `.systemLarge` widget.
- Header row: current year on the left; current streak count with a flame icon on the right (hidden if streak is 0).
- Main area: an 18-row × 21-column grid of small rounded-rectangle cells representing all 365 (or 366) days of the year in sequential order, left-to-right, top-to-bottom.
- Cell color encodes activity: no data → gray at 10% opacity, partial progress → `goalInProgress` at `max(0.2, progress)` opacity, full progress → `goalCompleted` (goal enabled) or `goalInProgress` solid (goal disabled).
- Footer row: days goal reached (with target icon in `goalCompleted` color) on the left; average steps per active day (with bar chart icon in `goalInProgress` color) on the right.
- Refreshes every 2 hours.

## Key Components
- **`YearGridLarge`** - Private view computing cell size dynamically via `GeometryReader` to fill available space. Spacing between cells is 2pt. Cells beyond day 365/366 render as `Color.clear`.
- **Cell fill** - `getFillColor(progress:)`: >= 1.0 → goal color, > 0 → `goalInProgress.opacity(max(0.2, progress))`, else → `gray.opacity(0.1)`.
- **Header** - Year as plain text + optional "N day streak" with `flame.fill` icon.
- **Footer** - Two `HStack` stat items: days reached and average steps.
- **Background** - `LinearGradient` from `goalInProgress` at 10% opacity (bottom-leading) to clear at 70% position.

## Data & Logic
- Data source: `Defaults[.stepDailyRecords]` filtered to `year == currentYear`.
- `dayProgresses: [Date: Double]` maps `startOfDay` dates to 0.0–1.0+ progress values.
- When `enableDailyStepGoal` is true: progress = `stepCount / historicalGoalForDay` (not capped, so overachievement is tracked but displayed as full color).
- When goal is disabled: progress = `stepCount / maxStepCountInYear` — relative scale.
- `daysReached`: count of days where progress >= 1.0.
- `averageSteps`: `totalSteps / yearRecords.count` (excludes days with no data entirely).
- `streakCount`: computed by `currentGoalStreak(records:goalLog:asOf:)` — walks backwards from today counting consecutive days where `goalAchievedWithLog(for:goalLog:)` is true.
- Year grid dates generated sequentially from Jan 1 to Dec 31 via `calendar.date(byAdding: .day, value: 1, to:)`.
- Widget kind: `"YearlyStepCalendarWidget"`.

## Premium
No gating — available to all users.

## Related Files
- `/Users/hieudinh/Projects/Steps/StepsWidget/YearlyStepCalendarWidget/YearlyStepCalendarWidget.swift`
- `/Users/hieudinh/Projects/Steps/StepsWidget/YearlyStepCalendarWidget/YearlyStepCalendarWidgetView.swift`

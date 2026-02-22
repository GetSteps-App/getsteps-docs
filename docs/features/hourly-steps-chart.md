# Hourly Steps Chart

## Overview
A bar chart showing step distribution across each hour of a selected day. Displayed on the home screen when the calendar view mode is set to Daily. Each bar represents one hour (0–23) and is color-coded based on whether the day's total steps meet the step goal.

## User Experience
1. Below the step count card, when in Daily mode, the hourly chart fills the chart area with 24 vertical bars.
2. The X-axis shows time labels every 6 hours (e.g., 0:00, 6:00, 12:00, 18:00).
3. The Y-axis shows step count values with grid lines.
4. All bars share the same color: green (`goalCompleted`) if the day's total steps meet or exceed the historical goal for that date, or blue (`goalInProgress`) if not.
5. When the user changes the selected date in the horizontal calendar, the chart animates to reflect the new hourly data.
6. There is no tap-to-select interaction on individual bars in the hourly chart. The weekly and monthly charts have tap selection; this chart is read-only.

## Key Components
- `HourlyStepsChartView` — the sole view. Accepts `hourlySteps: [HourlyStepData]`, `totalSteps: Int`, `date: Date`, `color: Color`, and an optional `height: CGFloat`.
- Uses Swift Charts `BarMark` with `x: .value("Hour", stepData.hour)` and `y: .value("Steps", stepData.steps)`.
- Color is determined by comparing `totalSteps` against `getGoalForDate(date, goalLog: goalChangeLog)` — uses the historical goal active on that specific date.
- X-axis marks use `AxisMarks(values: .stride(by: 6))` to show labels every 6 hours.
- Chart animates on `hourlySteps` array changes with `.animation(.default, value: hourlySteps)`.

## Data & Logic
- Data source: `dataStore.getHourlySteps(for: selectedDate)` returns `[HourlyStepData]`, where each entry has `hour: Int` (0–23) and `steps: Int`.
- `totalSteps` is sourced from `dataStore.stepDailyRecords.first(where: isDate inSameDayAs selectedDate)?.stepCount` (falls back to 0).
- `goalChangeLog` (`Defaults[.goalChangeLog]`) is an ordered log of `GoalChangeEntry` records. `getGoalForDate(_:goalLog:)` walks the log to find the goal that was active on the given date, supporting users who have changed their goal over time.
- Hourly data is fetched from HealthKit via `HKStatisticsCollectionQuery` with a 1-hour interval component (in `DataStore.fetchHourlyStep`).

## Premium
Not gated. Hourly chart is available in Daily mode, which is the free default mode.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Home/HourlyStepsChartView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/HourlyStepData.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/DataStore.swift`

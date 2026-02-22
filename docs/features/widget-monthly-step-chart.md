# Widget: Monthly Step Chart

## Overview
A medium-size home screen widget showing a bar chart of daily step counts for every day of the current month. Gives users a high-level view of their monthly activity pattern.

## User Experience
- Header shows current month and year (e.g., "Jun 2025"), total distance for the month on the left, and total step count on the right.
- Bar chart covers all days in the month (28–31 bars). X-axis labels appear on day 1 and every 5th day thereafter to avoid crowding. Y-axis labels use K-notation (e.g., "8K").
- Bar colors follow goal achievement: zero steps → gray 20%, goal achieved → `goalCompleted`, below goal → `goalInProgress`, goal disabled → always `goalInProgress`.
- Historical goal per day is respected (if the user changed their goal mid-month, each day is compared against the goal that was active on that day).
- Refreshes every 3 hours (monthly data changes slowly).

## Key Components
- **Header row** - Month + year (`.abbreviated` + year), total distance (converted to user unit), total steps.
- **`Chart`** over `ChartDataPoint` array - One `BarMark` per day, corner radius 2, color from `getBarColor(for:)`.
- **X-axis** - Day number labels, shown only for day 1 and multiples of 5.
- **Y-axis** - Grid lines with K-formatted step count labels.
- **`ChartDataPoint`** - Private struct holding `day` (String), `date`, `steps`, and `goal` (historical).
- **Background** - `RadialGradient` in `goalInProgress` at 10% opacity from bottom center.

## Data & Logic
- Data source: `Defaults[.stepDailyRecords]` filtered to the current month (`equalTo: Date(), toGranularity: .month`).
- Builds a dictionary `recordsByDay: [Int: StepDailyRecord]` keyed by calendar day number for O(1) lookup.
- Generates entries for all days in month using `calendar.range(of: .day, in: .month, for: Date())`.
- Historical goal fetched per day from `Defaults[.goalChangeLog]` via `getGoalForDate(_:goalLog:)`.
- `totalDistance` and `totalStep` are sums across all month records.
- Widget kind: `"MonthlyStepChartWidget"`.

## Premium
No gating — available to all users.

## Related Files
- `/Users/hieudinh/Projects/Steps/StepsWidget/MonthlyStepChartWidget/MonthlyStepChartWidget.swift`
- `/Users/hieudinh/Projects/Steps/StepsWidget/MonthlyStepChartWidget/MonthlyStepChartWidgetView.swift`

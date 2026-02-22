# Widget: Weekly Step Chart

## Overview
Displays a bar chart of daily step counts for the current week. Available in small and medium sizes. Helps users track their week-over-week consistency and see which days they hit their goal.

## User Experience
- Small size: Shows "This Week" label, total step count in large serif font, and a compact 7-bar chart with narrow weekday labels. Y-axis is hidden.
- Medium size: Adds a horizontal header with "This Week" label and total steps + "steps" unit. Y-axis is visible with K-formatted labels. A dashed goal line appears when the daily step goal is enabled.
- Bars for future days are shown as very faint gray. Bars for days with zero steps use a slightly darker gray. Days that met their historical goal show `goalCompleted` gradient; days below goal show `goalInProgress` gradient.
- Today's weekday label is bold; others are regular weight.
- Refreshes every 60 minutes during active hours (6am–11pm), every 2 hours at night.

## Key Components
- **`WeeklyStepChartSmallView`** - VStack with header (label + total steps) and a `Chart` with hidden Y-axis.
- **`WeeklyStepChartMediumView`** - HStack header + `Chart` with visible Y-axis grid lines and a `RuleMark` for the goal line.
- **`WeekDayStepData`** - Per-day model with `date`, `steps`, `goal` (historical for that day), `isToday`, `isFuture`.
- **Bar color logic** - Future days → gray 20%, zero steps → gray 30%, goal enabled → compare steps vs `day.goal` (historical goal for that day), no goal → always `goalInProgress`.
- **Goal line** - Dashed `RuleMark` at the current `stepGoal` value, only shown when `enableDailyStepGoal` is true.
- **Background** - `RadialGradient` in `goalInProgress` at 10% opacity from bottom center.

## Data & Logic
- Data source: `Defaults[.stepDailyRecords]` for the current week (Sunday or Monday start depending on `weekStartsOn` preference).
- Historical goal per day fetched from `Defaults[.goalChangeLog]` via shared `getGoalForDate(_:goalLog:)` helper — accounts for goal changes mid-week.
- `totalSteps` sums only non-future days.
- `maxSteps` is the max of the highest bar and the current `stepGoal`, ensuring the goal line never clips.
- Widget kind: `"WeeklyStepChartWidget"`.

## Premium
No gating — available to all users.

## Related Files
- `/Users/hieudinh/Projects/Steps/StepsWidget/WeeklyStepChartWidget/WeeklyStepChartWidget.swift`
- `/Users/hieudinh/Projects/Steps/StepsWidget/WeeklyStepChartWidget/WeeklyStepChartWidgetView.swift`

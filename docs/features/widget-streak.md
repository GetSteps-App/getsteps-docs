# Widget: Streak

## Overview
Displays the user's current consecutive-days-goal-met streak alongside a 7-day week view showing each day's achievement status, today's step count vs goal, and a progress gauge. Available in small and medium sizes. Requires the daily step goal to be enabled to show meaningful data.

## User Experience
- **Small size**: Flame icon with streak count at the top, 7 compact day icons below (narrow weekday labels), a divider, then today's step count vs goal text and a linear gauge.
- **Medium size**: Large step count / goal header on the left with streak count and flame icon on the right, a linear gauge below, then a 7-day row with abbreviated weekday labels and larger icons.
- Each day in the week row shows one of three states:
  - `achieved` — filled checkmark circle in `goalCompleted` gradient.
  - `currentIncomplete` — faint filled circle in `goalInProgress` gradient with a partial arc overlay showing the day's progress percentage.
  - `future` — quaternary filled circle.
- Today's label is bold; other days are regular weight.
- If `enableDailyStepGoal` is false, both sizes show a "Enable step goal to see your streak" placeholder with a flame outline icon.
- Background gradient shifts between `goalInProgress` and `goalCompleted` intensity based on today's progress.
- Refreshes every 1 hour.

## Key Components
- **`StreakWidgetView`** — Router that dispatches to `StreakWidgetSmallView` or `StreakWidgetMediumView` based on `widgetFamily`.
- **`StreakWidgetData`** — Business logic struct. Computed at init: resolves `stepGoal`, `progress`, `todayStatus`, and all 7 `weekDays: [StreakWeekDayData]` using historical goal log.
- **`StreakWeekDayData`** — Per-day model with `date`, `status: DayStatus`, `isToday`.
- **`DayStatus` enum** — `.achieved`, `.currentIncomplete(Double)`, `.future` with computed `iconName`, `foregroundStyle`, `opacity`.
- **Progress gauge** — SwiftUI `Gauge` (linear style) tinted with `todayStatus.foregroundStyle`.
- **Partial arc** — `Circle().trim(from:to:)` overlay on incomplete day icons, rotated -90° to start from top.

## Data & Logic
- Streak data source: `Defaults[.stepDailyRecords]` read by `StreakWidgetProvider`.
- `currentGoalStreak` walks backwards day-by-day from today (or yesterday if today is not yet achieved) counting consecutive days where `goalAchievedWithLog(for:goalLog:)` returns true.
- `goalAchievedWithLog` compares `record.stepCount >= goalForDay`, where `goalForDay` comes from `getGoalForDate(_:goalLog:)` using `Defaults[.goalChangeLog]`.
- Today's step count from `Defaults[.watchCurrentSteps]` / today's `StepDailyRecord`.
- Week start determined by `getFirstDayOfWeek(for:)` using `Defaults[.weekStartsOn]`.
- `progress = stepCount / todayGoal` (uncapped).
- Widget kind: `"StreakWidget"`.

## Premium
No gating — available to all users. However, streak tracking itself requires `enableDailyStepGoal` to be on; without it the widget shows a disabled placeholder.

## Related Files
- `/Users/hieudinh/Projects/Steps/StepsWidget/StreakWidget/StreakWidget.swift`
- `/Users/hieudinh/Projects/Steps/StepsWidget/StreakWidget/StreakWidgetView.swift`
- `/Users/hieudinh/Projects/Steps/StepsWidget/StreakWidget/StreakWidgetData.swift`

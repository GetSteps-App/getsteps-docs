# Widget: Today Step Count

## Overview
Displays today's step count in real-time with distance and calorie data. The primary home screen widget for the Steps app, supporting multiple sizes including Lock Screen and Dynamic Island complications.

## User Experience
- User adds the widget to Home Screen or Lock Screen via the standard iOS widget picker.
- Widget shows the current step count prominently, with distance and calories as secondary metrics.
- A progress ring (on `.systemSmall`) or border arc traces around the widget edge as steps accumulate toward the daily goal.
- The background gradient shifts from in-progress color to goal-completed color when the goal is reached.
- If the user has enabled a motivational message mode, a short encouraging phrase appears below the step count instead of showing calories separately.
- Widget refreshes every 15 minutes during active hours (7am–10pm) and every 60 minutes at night.

## Key Components
- **`.systemSmall`** - Large step count centered, distance above, calories below, progress arc border overlay with linear gradient background. Font is serif for the number.
- **`.accessoryRectangular`** (Lock Screen) - "Today steps" label, large step count on left, distance and calories stacked on right.
- **`.accessoryInline`** (Lock Screen inline) - Step count, distance, calories, and walk icon in a single line.
- **`.accessoryCircular`** (Lock Screen circular) - Circular gauge showing goal progress with shoe emoji and step count; falls back to plain count if goal is disabled.
- **Progress arc overlay** - Two `.trim` strokes on `ContainerRelativeShape` that together paint up to 360 degrees of the widget border, colored `goalInProgress` or `goalCompleted`.
- **Gradient background** - `LinearGradient` from `goalInProgress/goalCompleted` at bottom-leading to clear, intensity driven by progress ratio.

## Data & Logic
- Data source: `Defaults[.stepDailyRecords]` — an array of `StepDailyRecord` stored via the `Defaults` (SwiftDefaults) library in the shared App Group container.
- Filters for today using `Calendar.isDate(_:inSameDayAs:)`.
- Falls back to a zero-value `StepDailyRecord` when no record exists for today.
- `progress = stepCount / stepGoal` — capped at 1 for the arc, uncapped for color logic.
- Unit conversion: distance stored in kilometers, converted to user's preferred unit system (`imperial` = miles, `metric` = km) at display time.
- `homeInsightMode == .motivationalMessage` controls whether the `message` field from `StepDailyRecord.encouragingMessage` replaces the calories row.
- Timeline policy: single entry, refreshed at a scheduled future date (15 min active / 60 min night).

## Premium
No gating — this widget is available to all users regardless of subscription status.

## Related Files
- `/Users/hieudinh/Projects/Steps/StepsWidget/TodayStepCountWidget/TodayStepCountWidget.swift`
- `/Users/hieudinh/Projects/Steps/StepsWidget/TodayStepCountWidget/TodayStepCountWidgetView.swift`
- `/Users/hieudinh/Projects/Steps/StepsWidget/StepsWidgetBundle.swift`

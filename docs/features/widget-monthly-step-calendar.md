# Widget: Monthly Step Calendar

## Overview
A small home screen widget that renders the current month as a compact calendar grid where each day cell is filled with a color intensity representing step activity. Gives users an instant visual heat-map of their monthly consistency.

## User Experience
- Displayed as a `.systemSmall` widget.
- Shows a month/year header ("Jun 2025") followed by a 7-column grid of rounded-rectangle cells, one per calendar day.
- Each cell's fill color encodes activity level: fully transparent gray for days with no data, `goalInProgress` at proportional opacity for partial progress, solid `goalCompleted` (when goal is enabled) or solid `goalInProgress` (when goal is disabled) for 100% progress.
- Today's cell is outlined with a subtle orange-red stroke (`#EB4726` at 50% opacity).
- Empty cells before the first day and after the last day of the month are fully transparent (no color).
- Week start (Sunday vs Monday) follows the user's `weekStartsOn` preference.
- Background is a diagonal linear gradient from `goalInProgress` at 10% opacity (bottom-leading) to clear (top-trailing).
- Refreshes every 2 hours.

## Key Components
- **`CalendarView`** - Shared reusable component (used by both widget and watch MonthView). Accepts `date`, `dayProgresses: [Int: Double]`, `showHeader: Bool`, and `spacing: CGFloat`.
- **`LazyVGrid`** - 7 flexible columns with consistent spacing. Cells are square (`aspectRatio(1, contentMode: .fit)`).
- **Cell fill color** - Computed by `getFillColor(at:)`: progress >= 1 → solid goal color, 0 < progress < 1 → `goalInProgress.opacity(progress)`, no data → `gray.opacity(0.1)`.
- **Today outline** - `RoundedRectangle` stroke at `#EB4726` opacity 0.5, visible only on today's index.
- **Index calculation** - Grid indices 0..<28, 0..<35, or 0..<42 depending on month length + leading weekday offset.

## Data & Logic
- Data source: `Defaults[.stepDailyRecords]` filtered to the current month.
- `dayProgresses: [Int: Double]` maps calendar day number (1–31) to a 0.0–1.0 progress value.
- When `enableDailyStepGoal` is true: progress = `stepCount / historicalGoalForDay` (capped at 1.0), using `getGoalForDate(_:goalLog:)`.
- When goal is disabled: progress = `stepCount / maxStepCountInMonth` — relative scale so the best day is always shown at full intensity.
- Week start offset: Sunday → offset 0, Monday → offset 1; adjusts `weekdayIndex` of the first day accordingly.
- Widget kind: `"MonthlyStepCalendarWidget"`.

## Premium
No gating — available to all users.

## Related Files
- `/Users/hieudinh/Projects/Steps/StepsWidget/MonthlyStepCalendarWidget/MonthlyStepCalendarWidget.swift`
- `/Users/hieudinh/Projects/Steps/StepsWidget/MonthlyStepCalendarWidget/MonthlyStepCalendarWidgetView.swift`
- `/Users/hieudinh/Projects/Steps/StepsWidget/MonthlyStepCalendarWidget/CalendarView.swift`

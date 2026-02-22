# Weekly Steps Chart

## Overview
A bar chart showing daily step counts for each day in the selected week. Displayed on the home screen when calendar view mode is set to Weekly (Pro only). Each bar represents one day of the week and can be tapped to select that day, which updates the step count card to show that day's detailed stats.

## User Experience
1. Below the step count card in Weekly mode, 7 bars appear — one per day of the selected week.
2. Each bar is colored green (`goalCompleted`) if that day's steps met or exceeded the historical goal for that date, or blue (`goalInProgress`) otherwise.
3. Tapping a bar selects it: the selected bar renders at full opacity with a rounded rectangular stroke overlay, while all other bars dim to 30% opacity.
4. Tapping the same bar again deselects it, returning all bars to full opacity and clearing the card's chart date selection.
5. The X-axis labels show the day-of-month number (e.g., "15"). The selected day's label renders bold.
6. The Y-axis labels abbreviate large values with "K" (e.g., "8K" for 8000).
7. When `stepRecords` changes (e.g., a new week is navigated to), the chart rebuilds and animates with `.animation(.default)`.

## Key Components
- `WeeklyStepsChartView` — main chart view. Accepts `stepRecords: [StepDailyRecord]`, a `@Binding var chartDateSelection: Date?`, `color: Color`, and optional `height: CGFloat`.
- `WeeklyStepsChartData` — local `Identifiable` struct with fields `id: String` (index as string), `day: String` (day-of-month), `steps: Int`, `date: Date`.
- `@State private var chartData: [WeeklyStepsChartData]` — derived from `stepRecords` on appear and on `stepRecords` change.
- `@State private var chartSelectionIndex: Int?` — tracks which bar is selected locally; synced bidirectionally with `chartDateSelection` binding.
- `chartOverlay` with a transparent `Rectangle` captures tap gestures via `updateSelectedIndex(at:proxy:geometry:)`.
- `updateSelectedIndex` uses `ChartProxy.value(atX:)` to map a tap X position to the bar index string, then looks up the corresponding `stepRecords` entry. Only bars with `stepCount > 0` are selectable.

## Data & Logic
- `stepRecords` is populated by `dataStore.getDailyStepsForWeek(containing: selectedDate)`, which uses a weekly cache keyed by the first day of the week (`weekKey` format `"YYYY-MM-dd"`).
- Goal color per bar: `getGoalForDate(dataPoint.date, goalLog: goalChangeLog)` reads the historical goal active on each specific day from `Defaults[.goalChangeLog]`.
- `chartDateSelection` binding flows up to `StepCountContentView`, which uses it to override the heading and stats displayed on the card for the selected day.
- When `chartDateSelection` changes externally (e.g., card swipe clears it to `nil`), `chartSelectionIndex` is updated via `onChange(of: chartDateSelection)`.

## Premium
Requires Pro subscription. Weekly mode is gated — free users are redirected to the paywall when attempting to select it.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Home/WeeklyStepsChartView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/DataStore.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/StepDailyRecord.swift`

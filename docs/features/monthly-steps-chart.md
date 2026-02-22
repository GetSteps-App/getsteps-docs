# Monthly Steps Chart

## Overview
A bar chart showing daily step counts for every day in the selected month. Displayed on the home screen when calendar view mode is set to Monthly (Pro only). All 31 possible day positions are always represented — days with no data render as zero-height bars. Tapping a bar selects that specific day and updates the step count card with that day's stats.

## User Experience
1. Below the step count card in Monthly mode, up to 31 bars appear — one per calendar day of the selected month.
2. Each bar is colored green (`goalCompleted`) if that day's step count met or exceeded the historical goal for that specific date, or blue (`goalInProgress`) otherwise.
3. Days with no data (future days, or days with no HealthKit records) render as invisible zero-height bars — their X-axis slot is still reserved so the axis is always full-width.
4. Tapping a bar selects it: the selected bar renders at full opacity with a rounded rectangular stroke overlay, while all other bars dim to 30% opacity.
5. Tapping the same bar again deselects it, restoring all bars to full opacity and clearing the card's chart date selection.
6. The X-axis shows day numbers at positions 1 and every 5th day (1, 5, 10, 15, 20, 25, 30). The selected day's label renders bold.
7. The Y-axis abbreviates values with "K" for thousands.
8. The chart animates when `stepRecords` changes.

## Key Components
- `MonthlyStepsChartView` — main chart view. Accepts `stepRecords: [StepDailyRecord]`, a `@Binding var chartDateSelection: Date?`, `color: Color`, and optional `height: CGFloat`.
- `MonthlyStepsChartData` — local `Identifiable` struct with `id: String { day }`, `day: String` (day-of-month as string "1"–"31"), `steps: Int`, `date: Date?`.
- `let allDays` — a constant array of strings "1" through "31" used to pad `chartData` with zero-step entries for missing days.
- `@State private var chartData: [MonthlyStepsChartData]` — built from `stepRecords` then padded with zeros for all 31 positions.
- `@State private var chartSelectionDay: String?` — the selected day string; synced with `chartDateSelection` binding.
- `chartOverlay` captures taps via `updateSelectedIndex(at:proxy:geometry:)`. Uses `ChartProxy.value(atX:)` to get the day string at the tap position. Only days with `stepCount > 0` are selectable.
- `goalAchievedForDataPoint(_:)` — checks `dataPoint.date` against the historical goal log; returns `false` for zero/nil-date entries.

## Data & Logic
- `stepRecords` is populated by `dataStore.getDailyStepsForMonth(containing: selectedDate)`, which uses a monthly cache keyed by `"YYYY-MM"`.
- Missing day slots are filled at build time: after mapping `stepRecords` to `MonthlyStepsChartData`, every string in `allDays` not already in `chartData` gets a zero-step entry with `date: nil`.
- `chartDateSelection` binding flows up to `StepCountContentView` to override the card heading and stats.
- When `chartDateSelection` is cleared externally, `chartSelectionDay` resets to `nil` via `onChange(of: chartDateSelection)`.
- X-axis label visibility: shown when `intDay == 1` or `intDay % 5 == 0` (and `intDay != 1` for the modulo branch to avoid duplicating 1).

## Premium
Requires Pro subscription. Monthly mode is gated — free users are redirected to the paywall when attempting to select it.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Home/MonthlyStepsChartView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/DataStore.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/StepDailyRecord.swift`

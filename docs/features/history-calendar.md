# History Calendar

## Overview
A full-screen sheet showing historical step data in either a scrollable calendar grid or a chronological list. Accessible from the home screen toolbar (iOS 26+) or the History tab (pre-iOS 26). Supports switching between months and toggling between calendar and list view modes. Free users are limited to the 3 most recent months; Pro users see all available history.

## User Experience

### Opening
- On iOS 26+: tapped from the calendar toolbar button on the home screen; presented as a sheet.
- On pre-iOS 26: the History tab in the tab bar always shows this view.

### Calendar Mode
1. Months are displayed as full-grid calendar blocks stacked vertically in a `LazyVStack`, oldest at top, newest at bottom. Default scroll anchor is `.bottom` (most recent month visible first).
2. Each month block (`CalendarMonthView`) shows the month name and year as a header, weekday column letters, then a 7-column grid of day cells.
3. Each day cell is a `RoundedRectangle` (16pt corner radius) filled based on goal progress:
   - `goalCompleted` color when `progress >= 1` and `enableDailyStepGoal` is on.
   - `goalInProgress` color at `progress` opacity when `0 < progress < 1`.
   - `goalInProgress` (full) when `progress >= 1` but goal tracking is disabled.
   - `gray.opacity(0.1)` for days with no data.
4. Today's date cell number is **bold**. Future day numbers are `.secondary` colored and non-tappable.
5. Tapping a day cell selects it (primary stroke border overlay). A `DailyRecordRow` detail card animates in at the top of the scroll view, showing that day's steps, distance, calories, and goal ring. An X button dismisses it. Tapping the same cell again also deselects.
6. If the current month scrolls off-screen, a "Jump to latest ↓" floating button appears at the bottom; tapping it scrolls back to the current month.
7. Month jump: a month picker menu in the toolbar (iOS 26+) or a horizontal scroll of `MonthSelectionButton` pills (pre-iOS 26) lets the user jump directly to a specific month.

### List Mode
1. Records for the selected month are shown as a `LazyVStack` of `DailyRecordRow` items, sorted most-recent-first.
2. The navigation title updates to show the selected month/year.
3. A calendar icon in the toolbar (iOS 26+) or a month selector strip appears to change the displayed month.
4. Switching from calendar mode to list mode resets `selectedDate` to nil.

### View Mode Toggle
- iOS 26+: a menu picker in the toolbar switches between "Calendar" and "List" modes.
- Pre-iOS 26: a single icon button in the header bar toggles between modes with a symbol effect transition.

## Key Components
- `HistoryView` — root view. Owns `availableMonths: [Date]`, `selectedDate: Date?`, and reads `viewMode` from `Defaults[.historyViewMode]`.
- `CalendarMonthView` — renders a single month grid. Takes `date`, `dayProgresses: [Int: Double]`, and `selectedDate` binding. Computes `allDates` for the month, calculates weekday offset (respects `weekStartsOn`), and sizes the grid to 28, 35, or 42 cells.
- `DailyRecordRow` — reusable row showing a day number with goal progress ring, weekday/month label, step count, distance, and calories. Used both in calendar overlay and list mode.
- `MonthSelectionButton` — capsule pill for the pre-iOS 26 month selector. Filled primary background when selected, card background otherwise.
- `TrackableItem` — wrapper view that reports its frame via `ItemFramePreferenceKey` to detect when the current month scrolls off-screen.
- `ItemFramePreferenceKey` — `PreferenceKey` accumulating `[Int: CGRect]` frame data from `TrackableItem`s.
- `detectVisibility()` — checks whether the last month's frame is visible on screen to show/hide the "Jump to latest" button.

## Data & Logic
- `setupAvailableMonths()`: extracts unique months from `dataStore.stepDailyRecords` (excluding future dates), adds current month if absent, sorts ascending. Pro users get all months; free users get only the 3 most recent (`suffix(3)`).
- `calculateDayProgresses(for month:) -> [Int: Double]`: filters records for the month, maps each day number to `min(stepCount / historicalGoalForDay, 1.0)`. Uses `getGoalForDate(_:goalLog:)` per day.
- `filteredRecords` (list mode): filters `stepDailyRecords` to the selected month and sorts descending.
- `selectedMonth` is a `@Binding` from `NavigationManager.historySelectedMonth`, allowing external navigation (e.g., tapping a month from another screen) to scroll the history calendar.
- Weekday offset in `CalendarMonthView`: `calendar.component(.weekday, from: firstDayOfMonth) - 1 - offset` where `offset` is 0 for Sunday-start, 1 for Monday-start. Negative results wrap by adding 7.

## Premium
Free users see only the 3 most recent months of history. Pro users have access to all available months (back to Jan 1, 2025 or 12 months, whichever is earlier, based on the DataStore fetch range).

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/History/HistoryView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/History/CalendarMonthView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/History/DailyRecordRow.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/DataStore.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/StepDailyRecord.swift`

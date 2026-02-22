# Horizontal Calendar

## Overview
A horizontally scrollable strip at the bottom of the home screen used to select the active date, week, or month. The strip content and item format change based on the current `calendarViewMode`. Goal achievement indicators (small filled circles) appear on items where the step goal was met.

## User Experience
1. The strip renders at the bottom of the home screen, below the chart.
2. In **Daily** mode: each item shows an abbreviated weekday name (e.g., "Mon") and the day number. When the day is the 1st of a month, the month number is appended (e.g., "1/6"). 120 days back from today are shown.
3. In **Weekly** mode: each item shows the abbreviated month and a compact date range (e.g., "Jun 15-21"). 52 weeks back are shown.
4. In **Monthly** mode: each item shows the year and abbreviated month name. 12 months back are shown.
5. The selected item has a rounded-rectangle background at full scale; non-selected items show a 70%-scaled background at zero opacity (invisible but maintaining layout space).
6. When `enableDailyStepGoal` is on, a small filled circle in the top-right corner of an item indicates the goal was achieved for that period (daily: exact day comparison; weekly: sum of week steps vs. sum of weekly historical goals; monthly: sum of month steps vs. sum of daily historical goals across all days in that month).
7. Tapping an item sets `selectedDate` to that item's date, triggering a data fetch and chart/card update.
8. On appear, the strip auto-scrolls to the currently selected item (centered). It also auto-scrolls when `timeUnits` changes (e.g., after a mode switch).
9. A significant time change notification (e.g., midnight rollover) triggers `calculateTimeUnits` to rebuild the array so today's slot appears.

## Key Components
- `HorizontalCalendarView` — the sole view. Owns `@State private var timeUnits: [Date]`.
- `ScrollViewReader` + `ScrollView(.horizontal)` with `HStack` of tappable items.
- `calculateTimeUnits(calendarViewMode:)` — builds the `timeUnits` array:
  - Daily: 120 dates from 119 days ago to today.
  - Weekly: 52 week-start dates (using `getFirstDayOfWeek(for:)`), from 51 weeks ago.
  - Monthly: 12 month markers; current month uses today's date, past months use the last day of that month.
- `getLastTimeUnitsForDate(date:)` — returns the scroll anchor date for the current selection (for monthly mode, returns today if in current month; otherwise the last day of the month).
- `isSelectedTimeUnit(_:)` — comparison logic differs per mode: daily uses `isDate(_:inSameDayAs:)`, weekly compares first-day-of-week, monthly compares at `.month` granularity.
- `monthlyGoalAchieved(for:)` — sums step records for the month and compares to the sum of `getGoalForDate` for every calendar day in that month.
- `weeklyGoalAchieved(for:)` — sums step records for the week and compares to the sum of `getGoalForDate` for each of the 7 days.
- Goal dot: `Image(systemName: "circle.fill")`, 8×8pt, `goalCompleted` color at full opacity when selected, 50% opacity when not selected.

## Data & Logic
- `weekStartsOn` (`Defaults[.weekStartsOn]`, values `"Sunday"` / `"Monday"`) controls which day is the start of the week, used in `getFirstDayOfWeek(for:)`.
- `goalChangeLog` (`Defaults[.goalChangeLog]`) is read to evaluate historical goals per day.
- `enableDailyStepGoal` controls whether goal dots are shown at all.
- On `timeUnits` change: `proxy.scrollTo(timeUnit.localTimeFormat, anchor: .center)` with animation, then `dataStore.fetchStepsForDate(timeUnit)`.
- On `calendarViewMode` change: `calculateTimeUnits` rebuilds and resets the strip.

## Premium
Not directly gated. However, Weekly and Monthly modes (which change the strip's content) require Pro. In Daily mode the strip is fully available to all users.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Home/HorizontalCalendarView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/DataStore.swift`

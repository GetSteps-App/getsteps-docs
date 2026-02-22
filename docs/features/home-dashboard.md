# Home Dashboard

## Overview
The home screen is the primary view of the Steps app. It composes the step count card, a chart (hourly/weekly/monthly depending on the selected mode), and the horizontal date/period calendar into a single scrollable layout. On iOS 26+ it renders inside a `NavigationStack` with toolbar buttons; on earlier iOS versions it uses a custom `HeaderView` at the top.

## User Experience
1. On launch the user sees today's step count card at the top, a chart below it, and a horizontal calendar strip at the bottom.
2. The header area (top-left) shows either current weather condition with temperature, or a welcome/greeting message if weather is disabled.
3. A period picker (Daily / Weekly / Monthly) lets the user switch chart granularity. Selecting Weekly or Monthly while not a Pro subscriber triggers the paywall.
4. Tapping any day in the horizontal calendar fetches that day's data and updates both the card and the chart.
5. When the app returns to the foreground (`scenePhase == .active`) the welcome message and today's step data are refreshed automatically.
6. A significant time change notification (e.g., midnight rollover) resets `selectedDate` to today.
7. After a date's data is fetched, if the user has met their step goal today, an App Store review prompt is shown after a 2-second delay (production only).

## Key Components
- `HomeView` — root container; chooses between iOS 26+ `NavigationStack` layout and older `VStack` layout; owns `selectedDate` and `chartDateSelection` state; presents `HistoryView` and `SettingsView` as sheets via toolbar buttons (iOS 26+).
- `HeaderView` — pre-iOS 26 header bar; shows weather or welcome message on the left, and the period picker menu on the right.
- `HeaderLeadingView` — iOS 26+ overlay at top-left showing weather or welcome message.
- `StepCountCardView` — swiped card component (see step-count-card.md).
- `HourlyStepsChartView` / `WeeklyStepsChartView` / `MonthlyStepsChartView` — conditionally rendered based on `calendarViewMode`.
- `HorizontalCalendarView` — date/period strip at the bottom.
- `CalendarViewModeView` — standalone picker used in some contexts to change the period mode.

## Data & Logic
- `DataStore.shared` (`@Observable`) provides `stepCount`, `distance`, `calories`, `flightsClimbed`, `message`, `welcomeMessage`, and `stepDailyRecords`.
- `calendarViewMode` is persisted via `Defaults[.calendarViewMode]` (values: `.daily`, `.weekly`, `.monthly`).
- On scene becoming active: `dataStore.updateWelcomeMessage()` and `dataStore.fetchStepsForDate(selectedDate)` are called.
- `getAdaptiveChartHeight()` computes a chart height that adapts to screen size.
- `presentReviewIfNeeded()` triggers `requestReview()` when `enableDailyStepGoal` is on, the selected date is today, and `stepProgress >= 1`.

## Premium
- Weekly and monthly calendar view modes are gated behind Pro. Attempting to select them while not a Pro user shows `PaywallView` as a full-screen cover, and `calendarViewMode` is reset to `.daily` on startup for non-Pro users.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Home/HomeView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Home/HeaderView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Home/HeaderLeadingView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Home/CalendarViewModeView.swift`

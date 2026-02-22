# Widget: Hourly Step Chart

## Overview
A medium-size home screen widget that renders a bar chart of step counts broken down by hour for the current day. Lets users see at a glance which hours of the day they were most active.

## User Experience
- Displayed as a `.systemMedium` widget on the Home Screen.
- Header row shows today's date (abbreviated weekday + day + month), total distance on the left, and total step count on the right.
- Below the header is a full-width hourly bar chart covering all hours from midnight to the current hour.
- Bars animate with `.numericText` transitions as live data updates.
- Widget refreshes every 30 minutes during active hours (6am–11pm) and every 2 hours at night.

## Key Components
- **Date header** - Formatted as `"Mon, Jun 21"` style using `.dateTime.weekday(.abbreviated).day().month(.abbreviated)`.
- **Distance label** - Converted from stored kilometers to the user's unit system (km or mi), shown with `.secondary` style.
- **Step count** - Total daily steps, serif font, animates via `.numericText` content transition.
- **`HourlyStepsChartView`** - A reusable chart component passed `hourlySteps` array, `totalSteps`, date, and a fixed `Color.goalInProgress` tint. Height is unconstrained (`nil`) so it fills available space.
- **Background** - `RadialGradient` emanating from the bottom center in `goalCompleted` color at 10% opacity, fading to clear by radius 200.

## Data & Logic
- Data source: `Defaults[.stepDailyRecords]` filtered to today's record.
- Uses `StepDailyRecord.hourlySteps` — an array of per-hour step counts stored alongside the daily record.
- Falls back to a zero-record (`stepCount: 0`, `hourlySteps: []`) if no today record exists.
- Distance converted at render time using `unitSystem.convertedDistance(fromKilometers:)`.
- Timeline policy: one entry, refreshed at 30 min (active hours) or 2 hours (night) intervals.

## Premium
No gating — available to all users.

## Related Files
- `/Users/hieudinh/Projects/Steps/StepsWidget/HourlyStepChartWidget/HourlyStepChartWidget.swift`
- `/Users/hieudinh/Projects/Steps/StepsWidget/HourlyStepChartWidget/HourlyStepChartWidgetView.swift`

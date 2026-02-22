# Streaks

## Overview
The streaks system tracks how many consecutive days a user has met their daily step goal. The current streak count is cached on `DataStore` and displayed as a flame badge on the step count card. Tapping the badge opens a detailed sheet with streak statistics, a 30-day heatmap calendar, milestone trophy badges, and a personal best indicator.

## User Experience

### Flame Badge (Home Screen)
1. A capsule-shaped label showing a flame icon and the current streak count sits in the top-left corner of the step count card.
2. The flame is orange when the streak is 1 or more days; grey when 0.
3. Tapping the badge opens `StreaksSheetViewDetailed` as a bottom sheet.

### Streaks Detail Sheet
1. Sections appear with a slide-up animation on open.
2. **Hero section**: current milestone trophy badge (left) + large serif streak day count (right), with AVG/DAY and TOTAL stats below.
3. **Today's progress card**: shows either an in-progress bar ("X to go / Y / Z") with an animated shimmer fill, or a completed state ("Day N locked in") with a green checkmark when today's goal has been met.
4. **30-day heatmap**: a calendar grid of the past 30 days. Each cell is color-coded by step intensity (grey = no steps, blue gradient = partial, green gradient = goal reached). Tapping a cell selects it and overlays a `DailyRecordRow` detail card at the top.
5. **Next milestone card**: shows the locked (greyed-out) trophy for the next tier and how many days remain to unlock it. Hidden if all milestones are earned.
6. **Milestones row**: all 7 tiers displayed in a horizontal row — earned badges render in full color, unearned in grey.
7. **Personal best**: a single line showing the longest-ever streak and its date range in purple.
8. **Footer**: "How streaks work" link opens a secondary sheet with FAQ text.

### How Streaks Work FAQ
- A day counts toward a streak only after the step goal is reached.
- Missing a full day's goal breaks the streak.
- Past days are evaluated against the goal that was active on that specific day (historical goal log).
- Milestones at 7, 14, 30, 60, 100, 200, and 365 days.

## Key Components
- `StreaksSheetViewDetailed` — main sheet view. Owns `currentDetails`, `bestDetails`, `todaySteps`, `dailyGoal`, `goalCompleted`, `last30Days`, and animation states.
- `HeatmapDayRecord` — private struct: `date`, `steps`, `isInStreak`, `goalReached`, `isToday`.
- `HeatmapCellView` — renders a single heatmap cell. Color intensity is `0.15 + (steps/maxSteps) * 0.85`. Goal-reached cells use `goalCompleted` color; partial-step cells use `goalInProgress`; zero cells are `gray.opacity(0.1)`. Selected cell shows a 3pt primary stroke border.
- `PremiumBadge` — custom `Canvas`-drawn trophy badge. 7 tiers with unique icons: Bronze (shield+chevrons), Silver (5-point star medal), Gold (crown), Platinum (6-point hexstar), Emerald (laurel wreath), Ruby (faceted gem), Diamond (prismatic rhombus). Earned vs. unearned states have distinct color ramps.
- `PremiumBadgeTier` — enum with `days`, `label`, `name`, `color`, `isPrismatic`, and full color ramp properties. `forStreak(_:)` returns the highest tier earned; `nextTier(for:)` returns the next tier to unlock.
- `HowStreaksWorkView` — secondary sheet with FAQ text in a `NavigationStack`.
- `streakDisplayCount` — adds 1 to the stored streak count when today's goal is already complete, so the badge immediately reflects the new streak day.

## Data & Logic
- `DataStore.currentStreakCount` — cached `Int`, recalculated by `recalculateCurrentStreak()` on init and after every `stepDailyRecords` change.
- Streak definition: consecutive days (going back from yesterday) where `stepCount >= historicalGoalForDate`. Today is included only after its goal is met.
- `currentGoalStreakDetails(records:)` returns `(count, startDate, avgSteps)`.
- `bestGoalStreakDetails(records:)` returns `(count, startDate, endDate, avgSteps)`.
- `goalAchievedWithLog(for:goalLog:)` checks a record against the goal that was active on its date.
- Heatmap grid: 30 days back from today. Start offset computed from weekday of the first day (respects `weekStartsOn` preference — Sunday or Monday). Grid is padded to complete the last row.
- Week header symbols come from `DateFormatter().veryShortWeekdaySymbols`, reordered if `weekStartsOn == "Monday"`.
- `totalStepsInStreak` = `Int(currentDetails.avg) * currentDetails.count`.
- `rebuildSnapshots()` is called in `.task` on sheet open. Progress bar animation is deferred 300ms after sections appear.

## Premium
The streak feature itself (flame badge, count, detail sheet, milestones) is available to all users as long as `enableDailyStepGoal` is enabled in settings. No Pro gate on this feature.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Home/StreaksSheetViewDetailed.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Home/PremiumBadge.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Home/StepCountContentView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/DataStore.swift`

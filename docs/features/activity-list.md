# Activity List

## Overview
The Activity List is the primary screen for reviewing past workouts. It displays a chronologically grouped, filterable list of completed outdoor activities (walking, running, cycling, hiking). Non-activity workout types (swimming, strength training, etc.) are excluded from this view.

## User Experience
1. User opens the Activity tab. The app calls `dataStore.fetchRecentWorkouts()` on appear.
2. A list of past workouts is shown, grouped by month (e.g., "February 2026"), most recent first.
3. On iOS 26+, a toolbar filter menu (funnel icon) lets the user select a workout type. On earlier iOS, horizontally scrollable pill buttons appear at the top of the scroll view.
4. Tapping a filter narrows the list to that activity type (Running, Walking, Cycling, or Hiking). The active filter pill is highlighted in the primary color; inactive ones use the card background.
5. On iOS 26+, a "+" toolbar button opens a menu to start a new workout directly from the Activity tab, redirecting to the Workout tab.
6. Pull-to-refresh re-fetches workouts from HealthKit.
7. If no workouts exist, or none match the active filter, a type-specific empty state is shown (icon + title + subtitle).
8. Non-Pro users see only workouts from the current month and the two preceding months. A "Load more" button is shown below the list; tapping it opens the paywall.

## Key Components
- `ActivityView` - root view, owns filter state and `showPaywall` flag.
- `WorkoutTypeFilter` enum - `.all`, `.running`, `.walking`, `.cycling`, `.hiking`; each has a localized `displayName`.
- `WorkoutTypeFilterButton` - capsule pill button; selected state inverts foreground/background colors.
- `ActivityRowView` - individual card per workout (see activity-list.md companion file for row details).
- `EmptyStateView` - contextual empty state with per-type icon, title, and body text.
- Section headers - bold "Month Year" strings (e.g., "February 2026") that pin while scrolling (`pinnedViews: .sectionHeaders`).

## Data & Logic
- Source: `DataStore.workouts` array (fetched from HealthKit via `fetchRecentWorkouts()`).
- Filter: `filteredWorkouts` first restricts to `isDisplayedInActivity` types (walking/running/cycling/hiking), then applies the selected `WorkoutTypeFilter`.
- Grouping: `groupedWorkouts` groups by month start date, sorts groups descending, sorts workouts within each group descending by `startDate`.
- Pro gate: non-Pro users' group input is pre-filtered to remove workouts older than the start of the month two months ago (`calendar.date(byAdding: .month, value: -2, to: currentDate)`).
- Distance is converted from meters to the user's preferred unit system (`unitSystem.convertedDistance`).

## Premium
Non-Pro users are limited to the current month plus the previous two months of history. Older workouts are hidden, replaced by a "Load more" button that opens `PaywallView`.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Activity/ActivityView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Activity/ActivityRowView.swift`

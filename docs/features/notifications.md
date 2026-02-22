# Notifications

## Overview
The app delivers three categories of local push notifications: goal achievement alerts when the daily step goal is reached, milestone progress nudges at the halfway point, and workout completion summaries after a session ends. A fourth notification type — "Apps Unlocked" — fires when step-gated app blocking is lifted. All notifications require explicit user permission and respect a per-day deduplication guard.

## User Experience
- On first relevant trigger (e.g., goal reached while app is backgrounded), the system prompts for notification permission (alert + sound + badge).
- When the daily step goal is reached and the app is not in the foreground, a notification fires immediately with a randomly chosen congratulatory title (e.g., "Goal Crushed!", "On Fire!") and a body message that includes the step count and, if exceeded, how many bonus steps were walked.
- When the user reaches approximately 50% of their goal, a milestone notification fires with a "Keep Moving!" style title and remaining steps to go.
- When a workout session completes, a workout-type-specific notification fires (e.g., "Run Complete!", "Ride Complete!") with duration and distance in the body.
- When step-gated app blocking is lifted (goal met), an "Apps Unlocked!" notification fires.
- All notification content is randomized from a pool of 6 variants for variety.
- Tapping a goal notification shows a "View Steps" action button that foregrounds the app. Tapping a workout notification shows "View Workout".
- At midnight, stale notification tracking identifiers for previous days are cleaned up automatically.

## Key Components
- **`NotificationManager`** — Singleton (`shared`). Owns `UNUserNotificationCenter`. Exposes four `@MainActor async` scheduling methods and one sync reset method.
- **`NotificationPermissionManager`** — Singleton (`shared`). Tracks `authorizationStatus: UNAuthorizationStatus` as an `@Observable` property. When status changes, updates `Defaults[.enableNotifications]` and pushes the new setting to the watch via `WatchConnectivityManager.shared.sendNotificationSettingsToWatch()`.
- **Notification categories**:
  - `"GOAL_ACHIEVEMENT"` — action: `"VIEW_STEPS_ACTION"` (foreground).
  - `"WORKOUT_COMPLETION"` — action: `"VIEW_WORKOUT_ACTION"` (foreground). Includes `workoutId` in `userInfo`.
- **Deduplication**:
  - Goal: identifier `"goal-achievement-yyyy-MM-dd"` stored in `Defaults[.goalNotificationIdentifiers]`. One per calendar day.
  - Milestone: guarded by `Defaults[.milestoneNotificationLastSentDate]` — one per calendar day.
  - App unlock: guarded by `Defaults[.appLockUnlockNotificationSentDate]` — one per calendar day.
  - Workout: identifier `"workout-completion-<uuid>"` stored in `Defaults[.workoutNotificationIdentifiers]` — one per workout ID ever.
- **`resetDailyNotificationTracking()`** — Removes goal identifiers that don't match today's date string and clears stale `lastSentDate` defaults. Called at midnight by the app.

## Data & Logic
- `scheduleGoalAchievementNotification(stepCount:stepGoal:isAppActive:)`:
  - Guards: app not active, `enableNotifications`, `canSendNotifications`, not already sent today, `stepCount >= stepGoal`.
  - Content: random title from 6 options, body differs if `stepCount > stepGoal` (shows extra steps) vs exact.
- `scheduleMilestoneNotification(stepCount:isAppActive:)`:
  - Guards: app not active, `canSendNotifications`, not already sent today.
  - Body shows `stepCount`, `remainingSteps = stepGoal - stepCount`, and `progressPercentage`.
- `scheduleWorkoutCompletionNotification(workout:)`:
  - Guards: `enableNotifications`, `canSendNotifications`, not already sent for this workout UUID.
  - Body uses `durationText` (formatted via `DateComponentsFormatter`) and `distanceText` (km).
  - Returns `Bool` indicating whether notification was successfully scheduled.
- `scheduleAppUnlockNotification(stepCount:stepGoal:isAppActive:)`:
  - Guards: app not active, `enableNotifications`, `canSendNotifications`, not already sent today.
- Permission request: `UNUserNotificationCenter.requestAuthorization(options: [.alert, .sound, .badge])`.
- `canSendNotifications`: true only when `authorizationStatus == .authorized` (not `.provisional` or `.ephemeral`).
- `openAppSettings()`: opens `UIApplication.openSettingsURLString` for users who denied permission.

## Premium
No Pro gating. Notifications are available to all users, subject to system permission being granted.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Utilities/NotificationManager.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Utilities/NotificationPermissionManager.swift`

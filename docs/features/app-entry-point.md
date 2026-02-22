# App Entry Point

## Overview
`StepsApp.swift` is the `@main` entry point for the Steps iOS app. It initializes all global singletons, configures PostHog analytics, sets up the SwiftUI environment object graph, registers background tasks, handles push notification routing, handles deep links, and gates the main UI behind onboarding completion.

## User Experience
1. On first launch the user sees `OnboardingView` (HealthKit permission request, goal setup, etc.).
2. After onboarding completes (`hasCompletedOnboarding == true`) the app transitions to `MainTabView` with a fade+scale animation.
3. For non-Pro users, a paywall is shown automatically: once per app version, and thereafter at most once every 7 days. It is suppressed for Pro users.
4. After the paywall (or if it is skipped), a language suggestion sheet appears if the device locale matches a supported non-default language and has not been shown before.
5. After the language sheet, an App Lock onboarding sheet appears once if the feature has not been introduced yet. Accepting it opens `AppLockSettingsView`.
6. The tab bar (iOS 26+: fluid tab bar with `tabBarMinimizeBehavior(.onScrollDown)`; pre-iOS 26: standard `TabView`) contains:
   - **Steps** (house / shoeprints icon) → `HomeView`
   - **Activities** (flame icon) → `ActivityView`
   - **Insights** (sparkles icon) → `InsightsView`
   - **Workout** (dynamic icon from `WorkoutManager`) → `WorkoutHomeView` (iOS 26+ only, shown only when a workout configuration exists)
   - Pre-iOS 26 also has **History** (calendar) and **Settings** (gear) tabs.
7. Tapping the active Steps tab resets `selectedDate` to today. Tapping the active History tab (pre-iOS 26) resets the history calendar to the current month.
8. Deep links with scheme `steps://open?screen=WORKOUT_DETAIL&workoutId=<UUID>` navigate directly to `ActivityDetailView` for a specific workout via `NavigationManager.navigateToWorkout(_:)`.
9. A significant time change notification resets `selectedDate` to today and refreshes the welcome message.
10. On scene becoming active: weather is refreshed, app lock day-rollover is checked, and app lock unlock state is re-evaluated against current step count.

## Key Components
- `StepsApp` (`@main App`) — creates all shared manager instances as `@State`, calls `PurchaseManager.shared.configure()` and PostHog setup in `init()`, injects all managers into the environment via `.environment(...)`.
- `ContentView` — root view after app init. Reads `hasCompletedOnboarding`; shows `OnboardingView` or `MainTabView`. Owns paywall, language suggestion, and app lock onboarding presentation logic. Handles `onOpenURL` deep links.
- `MainTabView` — renders the `TabView` (iOS 26+ fluid tabs vs. pre-iOS 26 standard tabs). Owns `selectedDate: Date` shared across `HomeView` and `InsightsView`. Handles significant time changes.
- `AppDelegate` (`UIApplicationDelegate`) — registers background refresh task (`com.hieudinh.Steps.backgroundRefresh`) and workout notification task (`com.hieudinh.Steps.workoutNotification`). Handles remote notification registration/opt-in/opt-out. Routes notification taps to the correct tab or workout detail. Handles active workout session recovery on iOS 26+.
- `AppSceneDelegate` (`UIWindowSceneDelegate`) — applies `AppearanceHelper.shared.overrideDisplayMode()` as early as possible to prevent appearance flash on scene connection and activation.

## Global Singletons Injected into Environment
| Instance | Type | Purpose |
|---|---|---|
| `DataStore.shared` | `DataStore` | All step data and HealthKit fetching |
| `PurchaseManager.shared` | `PurchaseManager` | RevenueCat Pro subscription state |
| `WatchConnectivityManager.shared` | `WatchConnectivityManager` | Apple Watch data sync |
| `NotificationManager.shared` | `NotificationManager` | Local notification scheduling |
| `NotificationPermissionManager.shared` | `NotificationPermissionManager` | Notification permission requests |
| `LocationManager.shared` | `LocationManager` | Location + WeatherKit refresh |
| `NavigationManager.shared` | `NavigationManager` | Cross-screen navigation state |
| `AppLockManager.shared` | `AppLockManager` | Step-goal-based app lock feature |

## Background Tasks
- `com.hieudinh.Steps.backgroundRefresh` — fetches today's steps silently, reloads widgets, fires goal achievement notification if goal met, checks app lock unlock. Reschedules itself immediately on execution.
- `com.hieudinh.Steps.workoutNotification` — fetches recent workouts to trigger workout completion notifications.

## Analytics
- PostHog is initialized in `StepsApp.init()` with API key `phc_yLTEcGJ80gY5Ihm1pPwRJ8vC1ua82YGgCNF1qpdFJvl` and host `https://us.i.posthog.com`.
- A unique `userID` UUID is generated and stored in `Defaults[.userID]` on first launch.

## Paywall Logic
```
show if:
  (lastPaywallVersionShown != currentAppVersion)
  OR (lastPaywallShownAt is nil OR lastPaywallShownAt + 7 days <= now)
```
Always skipped for Pro users. After dismissal, notification permission is requested and subsequent sheets are triggered in sequence.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/StepsApp.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/DataStore.swift`

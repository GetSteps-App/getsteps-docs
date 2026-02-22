# App Lock

## Overview

App Lock uses Apple's Screen Time / Family Controls framework to block selected apps until the user reaches their daily step goal. Once the goal is met, shields are removed automatically and the user receives a notification. Shields reset at midnight via `DeviceActivity` monitoring. This is a Pro-only feature.

## User Experience

### Intro Sheet (`AppLockOnboardingView`)
- Presented as a medium-detent sheet when the feature is first promoted.
- Shows a `lock.app.dashed` icon, title "Introducing App Lock", and a scrolling row of social app icons (Twitter/X, Instagram, Reddit, TikTok, etc.) auto-scrolling via a 0.01 s timer.
- Description: "Take control of your screen time. Set daily limits for social apps and stay focused on what matters most."
- "Get Started" button dismisses the sheet and triggers the settings flow.

### Settings (`AppLockSettingsView`)
1. User opens Settings → "App Lock" row (shows "On" / "Off" status).
2. A sheet opens with an "Enable App Lock" toggle.
   - If user is not Pro, tapping the toggle opens `PaywallView` instead.
   - If Pro, enabling triggers `AuthorizationCenter.shared.requestAuthorization(for: .individual)` (Screen Time permission). A spinner shows during authorisation.
   - If permission denied, an alert explains that Screen Time must be enabled in iOS Settings.
3. When enabled, a "Select Apps" button appears below the toggle. Tapping it opens the system `FamilyActivityPicker`.
   - Selected apps and categories are listed below the button using their system labels.
4. An info label explains: "Selected apps will be blocked until you reach your daily step goal. Shields reset at midnight."
5. When disabled (not Pro-locked), three explanation rows describe the feature: "Choose Apps to Lock", "Walk to Unlock", "Resets Daily".
6. Disabling the toggle calls `AppLockManager.disable()` which removes shields, stops monitoring, and clears the unlocked-today flag.

### Runtime Behaviour
- On each step count update, `AppLockManager.checkAndUnlock(stepCount:goal:)` is called from `DataStore`.
- When `stepCount >= goal` and not yet unlocked today: shields are removed, `appLockUnlockedToday = true`, and an "Apps Unlocked!" notification is scheduled.
- If the user lowers their step goal mid-day below current steps, `onGoalChanged` re-evaluates and may unlock immediately.
- At midnight, `DeviceActivity` fires the extension which resets `appLockUnlockedToday = false` and re-applies shields.
- Step data (current steps + goal) is written to shared `UserDefaults` (App Group `group.com.hieudinh.Steps`) for use by the DeviceActivity and ShieldConfiguration extensions.

## Key Components

- `AppLockManager` — `@Observable @MainActor` singleton managing all Screen Time state:
  - `ManagedSettingsStore(named: "stepGoalLock")` — applies/removes app shields.
  - `FamilyActivitySelection` — persisted via `PropertyListEncoder` to App Group `UserDefaults`.
  - `DeviceActivityCenter` — schedules daily monitoring (00:00–23:59, repeating) for midnight reset.
  - `AuthorizationCenter` — requests and tracks FamilyControls authorisation status.
- `AppLockSettingsView` — settings sheet UI; manages paywall gate, authorisation flow, and app picker presentation.
- `AppLockOnboardingView` — introductory sheet with animated social app icon scroll row.
- `AppLockConstants` — shared string constants for store name, activity name, and `UserDefaults` keys used by both the main app and extensions.

## Data & Logic

- `@Default(.appLockEnabled)` — master on/off flag.
- `@Default(.appLockUnlockedToday)` — true once goal met for the day; reset at midnight.
- `@Default(.appLockLastResetDate)` — tracks last midnight reset date to handle app restarts.
- `sharedDefaults` — `UserDefaults(suiteName: "group.com.hieudinh.Steps")` shared with extensions.
- Keys written to shared defaults: `appLockCurrentSteps`, `appLockStepGoal`, `appLockEnabled`, `appLockUnlockedToday`, `appLockSelection`.
- Selection persistence: encoded with `PropertyListEncoder`; decoded on init via `loadPersistedSelection()`.
- `checkDayRollover()` called on `AppLockManager.init` to handle cases where the app was not running at midnight.
- Shield application: `store.shield.applications` set to `selection.applicationTokens`; `store.shield.applicationCategories` set to `.specific(selection.categoryTokens)`. Only applied when both authorised and not yet unlocked today.

## Premium

App Lock **requires Pro subscription**. The toggle is gated — non-Pro users tapping "Enable App Lock" are redirected to `PaywallView`. The `AppLockPaywallView` is shown as one of the 8 feature carousel cards in the paywall.

## Related Files

- `/Users/hieudinh/Projects/Steps/Steps/AppLock/AppLockManager.swift`
- `/Users/hieudinh/Projects/Steps/Steps/AppLock/AppLockSettingsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/AppLock/AppLockOnboardingView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/AppLock/AppLockConstants.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Settings/SettingsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Paywall/AppLockPaywallView.swift`

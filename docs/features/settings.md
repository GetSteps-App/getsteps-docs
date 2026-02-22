# Settings

## Overview

The Settings screen is the central configuration hub for the Steps app. It is presented as a sheet from any tab. Non-Pro users see a "Get Steps Pro" upgrade card at the top. Settings are grouped into three sections: App Settings, Support, and About.

## User Experience

1. The sheet opens from a gear icon (iOS 26+) or a settings button elsewhere in the app.
2. Non-Pro users see a gradient upgrade card with a "Start Your Free Trial" CTA that opens `PaywallView`.
3. **App Settings section:**
   - **Appearance** — menu picker: System / Light / Dark. Change applies immediately via `AppearanceHelper.shared.overrideDisplayMode()`.
   - **Color Theme** — opens `ColorThemeSettingView` sheet; shows current theme name and two colour swatches.
   - **Unit system** — menu picker: Metric / Imperial. Triggers widget reload and sends update to Apple Watch.
   - **Week starts on** — menu picker: Sunday / Monday. Triggers widget reload and Watch sync.
   - **View steps by** — menu picker: Daily / Weekly (Pro) / Monthly (Pro). Non-Pro tapping a non-daily option triggers `PaywallView`.
   - **Daily goal** — navigates to `GoalSettingView` sheet; shows current goal value if enabled.
   - **App Lock** — navigates to `AppLockSettingsView` sheet; shows "On" / "Off" status.
   - **Notifications** — toggle. Enabling requests permission via `NotificationPermissionManager`; if denied, opens iOS Settings. Disabling unregisters remote notifications and calls the opt-out API.
   - **Home insight** — menu picker controlling what metric the home screen insight card displays; triggers widget reload.
   - **Show weather** — toggle. Enabling requests location permission; disabling clears cached location and weather data.
   - **App icon** — opens `AppIconSelectorView` sheet; shows current icon name.
   - **Home Screen widgets** — opens `HomeScreenWidgetSettingView` sheet.
   - **App Language** — opens iOS Settings (via `UIApplication.openSettingsURLString`); shows current locale language name.
4. **Support section:**
   - **Feedback Board** — opens `AllFeatureRequestsView` sheet.
   - **Leave a Review** — opens App Store review URL.
   - **Restart Onboarding** — shows confirmation alert; on confirm sets `hasCompletedOnboarding = false` and resets TipKit datastore.
   - **Re-sync Apple Health data** — clears cached step records and workouts, re-fetches from HealthKit; shows spinner during sync and a success toast on completion.
   - **Manage Subscription** — shown only when `purchaseManager.managementURL` is non-nil; opens subscription management URL.
   - **Terms of Service** — opens `https://getsteps.app/terms`.
   - **Privacy Policy** — opens `https://getsteps.app/privacy`.
   - **Fitness Calculators** — opens `https://getsteps.app/tools`.
5. **About section:**
   - App name and version string.
   - **Credits** — opens `CreditsView` sheet.
   - **Share Steps** — opens `QRCodeShareSheet` at height 500 with thin material background.
   - **Follow me on X** — opens `https://x.com/hieudinh_`.
   - **Blog** — opens `https://getsteps.app/blog`.
   - **View Plans** — shown only to Pro users; opens `PaywallView` to review current plan.

## Key Components

- `SettingsView` — root scroll view with all sections; manages 13+ local `@State` flags for sheets and alerts.
- `goProCard` — green-tinted card with a gradient "Steps Pro" label, shown only to non-Pro users.
- Each setting row uses a `cardStyle` modifier for consistent card appearance.
- iOS 26+ uses `NavigationStack` with a `.cancellationAction` close button; earlier iOS uses a variable-blur overlay with an `xmark` button.

## Data & Logic

- `@Default` bindings (via `Defaults` library): `appearanceMode`, `unitSystem`, `stepGoal`, `enableDailyStepGoal`, `weekStartsOn`, `stepCountMode`, `homeInsightMode`, `enableNotifications`, `showWeather`, `calendarViewMode`, `colorThemePreset`, `appLockEnabled`.
- `PurchaseManager.isProUser` gates the upgrade card and `calendarViewMode` non-daily options.
- `WatchConnectivityManager.sendStepGoalToWatch(...)` is called whenever `unitSystem`, `weekStartsOn`, `stepCountMode`, or `enableDailyStepGoal` changes.
- `WidgetCenter.shared.reloadAllTimelines()` called on unit system, week start, home insight, and calendar view mode changes.
- Re-sync: clears `Defaults[.stepDailyRecords]` and `Defaults[.workouts]`, then calls `fetchMissingStepRecordsIfNeeded` and `fetchRecentWorkouts` in a `DispatchGroup`; on completion calls `fetchStepsForDate` and shows a toast.
- Weather toggle: if location denied/restricted, toggle is set back to false; if undetermined, requests permission; if authorised, requests current location.
- App icon initialisation: reads `UIApplication.shared.alternateIconName` on appear to set `selectedAppIcon`.

## Premium

- **Weekly/Monthly step view** (calendarViewMode) requires Pro.
- The upgrade card is shown to non-Pro users; Pro users see "View Plans" instead.
- All other settings rows are free.

## Related Files

- `/Users/hieudinh/Projects/Steps/Steps/Settings/SettingsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Settings/GoalSettingView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Settings/AppIconSelectorView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Settings/ColorThemeSettingView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Settings/QRCodeShareSheet.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Settings/HomeScreenWidgetSettingView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Settings/CreditsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Settings/AllFeatureRequestsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/AppLock/AppLockSettingsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Paywall/PaywallView.swift`

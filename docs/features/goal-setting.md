# Goal Setting

## Overview

Goal Setting lets the user configure an optional daily step target. It is accessible from Settings ("Daily goal" row) and also embedded in the Onboarding step 2 flow. The feature includes a toggle to enable/disable the goal, a wheel picker to choose the target value, an estimated distance preview, and a milestone card that provides motivational feedback based on the chosen goal.

## User Experience

1. User opens Settings and taps "Daily goal". A sheet appears (`GoalSettingView`).
2. An "Enable Daily Goal" toggle row appears at the top. When off, the rest of the UI is hidden with an animated transition.
3. When enabled, three additional sections appear:
   - **Current Goal display** — large bold number with "steps per day" label, animates with `contentTransition(.numericText())`.
   - **Adjust Your Goal** — a wheel `Picker` with values from 1,000 to 20,000 in steps of 500. Scrolling immediately updates the goal and syncs to Apple Watch.
   - **Estimated Distance** — calculated as `selectedGoal × averageDistancePerStep()`, converted to the user's unit system (km or mi), shown with 2 decimal places.
4. A **milestone card** below the picker shows a contextual icon, title, and description that changes based on the selected goal range:
   - 1,000–2,999: "Getting Started! 🌱" / leaf icon / green
   - 3,000–4,999: "Building Momentum! 💪" / bolt icon / blue
   - 5,000–7,999: "Good Daily Habit! ⭐" / star icon / purple
   - 8,000–9,999: "Active Lifestyle! 🚀" / paperplane icon / orange
   - 10,000–12,999: "Excellent Goal! 🎯" / target icon / yellow
   - 13,000–15,999: "High Achiever! 🏆" / trophy icon / pink
   - 16,000–20,000: "Athletic Performance! 🥇" / medal icon / red
5. On dismiss, if the goal or enable-state changed: `dataStore.stepProgress` is recalculated, encouraging messages update, widgets reload, and App Lock re-evaluates unlock status if the goal was lowered below current steps.

## Key Components

- `GoalSettingView` — full sheet view; contains toggle, goal picker, distance preview, and milestone card.
- Enable toggle — `Toggle` bound to `$enableDailyStepGoal`; sends Watch sync on change.
- Wheel picker — `Picker` with `.wheel` style; `goalRange` is `stride(from: 1000, through: 20000, by: 500)`.
- Milestone card — `VStack` with `contentTransition(.numericText())` and `.animation(.default, value: selectedGoal)`; background uses milestone colour at 10% fill with 30% stroke border.
- iOS 26+ uses `NavigationStack` + `.cancellationAction` toolbar button; earlier iOS uses variable-blur overlay with an `xmark` button.

## Data & Logic

- `@Default(.stepGoal)` and `@Default(.enableDailyStepGoal)` persist via the `Defaults` library.
- `selectedGoal` is a local `@State` initialised to `stepGoal` on `.task`; picker changes write both `selectedGoal` and `stepGoal` simultaneously.
- `initialStepGoal` / `initialEnableDailyStepGoal` are captured on appear to detect actual changes on dismiss.
- `averageDistancePerStep()` is a utility function (defined outside the view) used for the estimated distance calculation.
- `logGoalChange(newGoal:)` records the goal change for streak preservation logic.
- Apple Watch sync: `watchConnectivityManager.sendStepGoalToWatch(_:enableDailyStepGoal:unitSystem:weekStartsOn:stepCountMode:)` called on picker change and toggle change.
- Widget reload: `WidgetCenter.shared.reloadAllTimelines()` called on dismiss if anything changed.
- App Lock: `AppLockManager.shared.onGoalChanged(newGoal:currentSteps:)` called if goal changed and App Lock is enabled.

## Premium

This feature is **free**. No Pro subscription required.

## Related Files

- `/Users/hieudinh/Projects/Steps/Steps/Settings/GoalSettingView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Settings/SettingsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Onboarding/OnboardingStep2View.swift`

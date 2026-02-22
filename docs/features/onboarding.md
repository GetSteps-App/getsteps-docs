# Onboarding

## Overview

Onboarding is a two-step first-launch flow that introduces the app, lets the user set an optional daily step goal, and requests HealthKit permission. It is shown whenever `hasCompletedOnboarding` is false (including after a "Restart Onboarding" from Settings). The flow uses animated emoji rows, coloured blurred circle backgrounds, and smooth slide/scale transitions between steps.

## User Experience

### Step 1 — Welcome (`OnboardingStep1View`)

1. App launches and shows a full-screen view with the app's background colour.
2. "Welcome to / Steps" fades in smoothly over ~2 seconds (serif font for "Steps").
3. After the welcome text fades, it transitions away and three infinite horizontal emoji-scroll rows animate in from the bottom (staggered: 0.3 s, 0.4 s, 0.5 s). The emoji pool contains 52+ activity emojis (walkers, runners, cyclists, swimmers, yoga, etc.) shuffled into three independent rows. Rows 1 and 3 scroll right; row 2 scrolls left; driven by a 0.01 s timer incrementing `contentOffset.x`.
4. Below the emoji rows, a headline slides up: "Your daily / **10,000** steps / journey starts today." with a subtitle.
5. A "Get Started →" button slides up last (0.8 s delay). Tapping cancels all timers and transitions to step 2.
6. Two blurred circle backgrounds (green and orange) drift randomly every 5 seconds via a background timer.

### Step 2 — Goal & Permissions (`OnboardingStep2View`)

1. Title and description change dynamically based on permission state (`.notRequested` → `.requesting` → `.granted` / `.denied`).
2. When permission not yet requested, the UI shows:
   - A "Set Daily Step Goal" toggle.
   - If enabled: a wheel picker (1,000–20,000 in 500-step increments) labelled "Choose Your Goal".
3. User taps "Continue" → `DataStore.shared.requestAuthorization` is called → permission dialog appears.
   - On grant: state changes to `.granted`; UI replaces the goal picker with two confirmation rows ("Private & Secure" and "All Set!").
   - On denial: state changes to `.denied`; a "Continue without Health Access" secondary button appears.
4. From `.granted` state, tapping "Start Tracking" (with a 1.5 s async delay and a spinner) calls `onComplete()` which sets `hasCompletedOnboarding = true`.
5. On completion: `logGoalChange(newGoal:)` logs the initial goal, `dataStore.stepProgress` is recalculated, encouraging messages update, and widgets reload.
6. A drifting orange blurred circle background animates throughout step 2.

## Key Components

- `OnboardingView` — root coordinator; holds `currentStep` state (0 or 1); applies `.scale(0.95).opacity` transitions between steps.
- `OnboardingStep1View` — welcome + emoji animation screen. Uses three `InfiniteScrollView` instances; `EmojiView` renders each emoji (iOS 26+: glass effect circle; earlier: outlined circle).
- `OnboardingStep2View` — goal + permission screen. `HealthKitPermissionStatus` enum drives title, description, and button state. `PrivacyInfoRow` displays confirmation rows after permission grant.
- `InfiniteScrollView` — generic infinite-scroll component (defined in `Onboarding/InfiniteScrollView.swift`); uses a `UIScrollView` bridge for direct `contentOffset` manipulation.
- `EmojiView` — renders a single emoji in a circle (glass on iOS 26+, stroked border on earlier).

## Data & Logic

- `@Default(.hasCompletedOnboarding)` — flipped to `true` to exit the flow; flipped back to `false` by "Restart Onboarding" in Settings.
- `@Default(.stepGoal)` and `@Default(.enableDailyStepGoal)` — written directly by the picker/toggle in step 2.
- `DataStore.shared.requestAuthorization(completion:)` — requests HealthKit read permissions; returns `Bool` success on main queue.
- `logGoalChange(newGoal:)` — utility function recording goal history for streak preservation.
- `WidgetCenter.shared.reloadAllTimelines()` — called on onboarding completion.
- `String: Identifiable` retroactive conformance (defined in `OnboardingStep1View.swift`) allows string arrays in `ForEach`.

## Premium

Onboarding is **free** and has no Pro gate.

## Related Files

- `/Users/hieudinh/Projects/Steps/Steps/Onboarding/OnboardingView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Onboarding/OnboardingStep1View.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Onboarding/OnboardingStep2View.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Onboarding/InfiniteScrollView.swift`

# Color Themes

## Overview

Color Themes lets users personalise the colour of their step goal progress indicators throughout the app (progress rings, bars, calendar dots, widgets). Users choose from a set of named preset palettes or define fully custom colours using a system colour picker. Changes apply immediately and reload all home screen widgets.

## User Experience

1. User opens Settings and taps "Color Theme". A sheet appears (`ColorThemeSettingView`).
2. The sheet shows a title ("Choose a color theme"), a subtitle, and a vertical list of preset rows plus a "Custom" row.
3. Each preset row shows two coloured circles (in-progress colour + completed colour), the preset name, and a checkmark when selected.
4. Tapping any preset immediately sets `colorThemePreset` and reloads widgets.
5. The "Custom" row, when selected, expands inline to reveal two `ColorPicker` controls — one for "In Progress" and one for "Completed". Changes to either picker update the persisted hex strings and reload widgets in real time.
6. iOS 26+ uses `NavigationStack` with a "Color Theme" navigation title and `.cancellationAction` close button; earlier iOS uses a variable-blur overlay `xmark` button.

## Key Components

- `ColorThemeSettingView` — main sheet view; owns the preset list and custom picker expansion logic.
- `presetRow(_:)` — a button row showing `swatchPair` + name + optional checkmark; tapping selects the preset.
- `customRow` — button row that also conditionally expands to show two inline `ColorPicker` controls when `.custom` is selected.
- `swatchPair(inProgress:completed:)` — a small `HStack` of two 20pt circles showing the two colours.
- `ColorThemePreset` enum — defines named presets (`presets` static array) each with `inProgressColor`, `completedColor`, and `displayName`. Also has a `.custom` case.

## Data & Logic

- `@Default(.colorThemePreset)` — persists the selected preset enum value.
- `@Default(.customInProgressColorHex)` — persists the custom in-progress colour as a hex string (e.g. "FF5500").
- `@Default(.customCompletedColorHex)` — persists the custom completed colour as a hex string.
- `customInProgress` and `customCompleted` are local `@State Color` values initialised from the persisted hex on appear.
- `Color.hexString` computed property (private extension): converts `UIColor` RGBA components to a 6-character uppercase hex string.
- `WidgetCenter.shared.reloadAllTimelines()` is called on every selection change and on every colour picker change.
- The `Color(hex:)` initialiser (defined elsewhere in the app) is used to reconstruct colours from stored hex strings.

## Premium

Color Themes requires **Pro subscription**. The `ColorThemePaywallView` in the paywall feature carousel promotes this feature. The settings row is accessible from Settings without a gate, but the paywall is shown when a non-Pro user attempts to use a non-default theme (gate logic is in `PurchaseManager` / checked at the point of use in the app's theme application layer).

## Related Files

- `/Users/hieudinh/Projects/Steps/Steps/Settings/ColorThemeSettingView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Settings/SettingsView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Paywall/ColorThemePaywallView.swift`

# App Icon Selector

## Overview

The App Icon Selector lets users change the home screen icon of the Steps app to one of several alternate icons. It is presented as a sheet from the Settings screen. Icons are displayed in a 4-column grid; tapping one applies it immediately via `UIApplication.setAlternateIconName`.

## User Experience

1. User opens Settings and taps the "App icon" row. The current icon name is shown as the row's trailing value.
2. A sheet slides up showing a title ("Choose your app icon"), a subtitle, and a 4-column `LazyVGrid` of all available icons.
3. Each icon is shown as a 64×64 rounded rectangle (cornerRadius 16). The currently selected icon has a blue 3pt border and is scaled up 1.05x with a spring animation.
4. Tapping a different icon calls `UIApplication.shared.setAlternateIconName` asynchronously. During the change, all buttons are disabled (`isChanging = true`).
5. On success, `selectedAppIcon` updates and the blue selection border moves to the new icon. On failure, an alert shows the error message.
6. Below the grid, a "Credits & Disclaimer" section attributes icons to Charlie Clark / thiings.co and includes a Nike trademark disclaimer.
7. iOS 26+ uses `NavigationStack` with a `.cancellationAction` close button; earlier iOS uses a variable-blur overlay `xmark` button.

## Key Components

- `AppIconSelectorView` — grid-based icon picker sheet.
- `AppIcon` enum — defines all available icon cases; each case has:
  - `.icon` / `.icon(for:)` — returns the `Image` asset for light or dark scheme.
  - `.name` — the alternate icon name string passed to `UIApplication`.
  - `.description` — display label shown below each icon in the grid.
- `LazyVGrid` — 4 flexible columns with 16pt spacing.
- Selection state: `@Binding var selectedAppIcon` shared back to `SettingsView` to update the settings row's trailing label.
- Error alert: shown when `UIApplication.setAlternateIconName` returns an error.

## Data & Logic

- `UIApplication.shared.setAlternateIconName(appIcon.name)` is called on the main queue inside `DispatchQueue.main.async`.
- The default icon (no alternate) uses `appIcon.name == nil`, which tells the system to revert to the primary icon.
- Current icon is initialised in `SettingsView.onAppear` by reading `UIApplication.shared.alternateIconName` and mapping it back to an `AppIcon` case via `AppIcon(from:)`.
- No persistence beyond the OS-level alternate icon setting; the `selectedAppIcon` binding is transient UI state.

## Premium

This feature is **free**. No Pro subscription required.

## Related Files

- `/Users/hieudinh/Projects/Steps/Steps/Settings/AppIconSelectorView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Settings/SettingsView.swift`

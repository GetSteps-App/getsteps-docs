# QR Code Sharing

## Overview

QR Code Sharing lets users share the Steps app with others via a scannable QR code or a direct App Store link. It is presented as a half-height sheet from the Settings "Share Steps" row. The QR code is generated at high resolution (1024pt × screen scale, 300 DPI) with the current app icon embedded in the centre.

## User Experience

1. User taps "Share Steps" in Settings. A sheet appears at a fixed height of 500pt with a thin material background.
2. The sheet shows a "Scan to download Steps" headline.
3. Below is a 300×300pt white rounded card displaying the generated QR code. While generating, a `ProgressView` spinner is shown.
4. Two action buttons sit below the QR code:
   - **Copy QR Code** — copies the generated `UIImage` to `UIPasteboard.general.image`. A "Image copied" toast appears.
   - **Copy Link** — copies the App Store URL string to `UIPasteboard.general.string`. A "Link copied" toast appears.
5. The QR code regenerates automatically when the device colour scheme changes (dark/light mode).
6. A rounded border stroke (secondary at 50% opacity) outlines the entire sheet.

## Key Components

- `QRCodeShareSheet` — the full sheet view; owns QR image state and both copy actions.
- QR generation (`generateQRCode(from:)`):
  - Uses the `QRCode` library builder API.
  - Quiet zone: 3 pixels.
  - Pixel shape: `Circle`.
  - Eye shape: `Circle`.
  - Logo: the selected app icon centred in the QR (`.circleCenter(inset: 0)`).
  - Foreground/background colours adapt to `colorScheme` (white-on-black in dark mode, black-on-white in light mode).
  - Output: PNG at `1024 * UIScreen.main.scale` dimension, 300 DPI.
- `selectedAppIcon: AppIcon` — passed from `SettingsView` so the correct icon asset is embedded in the QR logo.
- Toast modifier — `.toast(isPresented:title:)` used on the button container; incremented integer pattern triggers the toast.

## Data & Logic

- `appURL` is `"https://apps.apple.com/us/app/steps-workout-pedometer/id6746096378"` (passed in from `SettingsView`).
- QR generation runs synchronously on appear and on `onChange(of: colorScheme)`.
- `onCopyImage` and `onCopyLink` are callbacks injected by `SettingsView`; actual clipboard writes happen in `SettingsView` (`copyImageToClipboard` / `copyTextToClipboard`).
- No network calls; QR is fully local.

## Premium

This feature is **free**. No Pro subscription required.

## Related Files

- `/Users/hieudinh/Projects/Steps/Steps/Settings/QRCodeShareSheet.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Settings/SettingsView.swift`

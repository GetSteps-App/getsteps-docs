# Activity Detail

## Overview
A full-screen detail view for a single completed workout. Shows a live map with the route polyline, a stats bottom sheet with all workout metrics, and an animated route playback mode. Accessed by tapping any row in the Activity List.

## User Experience
1. User taps a workout row; the detail opens as a full-screen cover (`fullScreenCover`).
2. The map fills the screen. While the route loads, a centered progress spinner appears over a blurred map background.
3. Once loaded, the route polyline is drawn in the workout's color. The map camera auto-positions to show the full route, shifted slightly north so it clears the bottom sheet.
4. The stats bottom sheet slides up from the bottom. It uses a glass material on iOS 26+, or `ultraThinMaterial` on earlier iOS.
5. The sheet header shows: workout type icon (colored circle), activity name with optional Indoor/Outdoor prefix, date/time, and weather data (temperature + humidity) if recorded.
6. A 2-column stats grid shows all available metrics.
7. Tapping a locked metric (shown with a particle spoiler effect) reveals nothing — the spoiler requires Pro to unlock.
8. If the workout has a route and the user is Pro, a Play button appears in the toolbar. Tapping it enters **playback mode**:
   - The stats sheet slides off screen; an elevation profile overlay slides up from the bottom.
   - The map zooms to show the full route.
   - A moving dot traces the route in real time over 10–30 seconds (scaled to distance).
   - A faded full route acts as a background reference.
   - Start (green dot) and end (red dot, shown only when complete) markers appear.
   - The elevation chart shows a filled gradient area for the completed portion and a current-altitude indicator dot.
   - Pause/Resume and Reset buttons appear in the toolbar.
   - After playback finishes, it auto-resets after 1 second.
9. A Share button (orange, in the toolbar) opens the sharing sheet (`ActivitySharePreviewSheet`).
10. If route data cannot be fetched, an error state with a Retry button is shown on the map.
11. Non-Pro users see "Unlock Steps Pro to see route details" and a `ProChipView` instead of the map route.

## Key Components
- `ActivityDetailView` - root view; owns all playback, route, and sharing state.
- Map (`Map` + `MapPolyline`) - displays the full route in normal mode and a progressive polyline in playback mode.
- `statsBottomSheet` - contains header, stats grid, and paywall prompts; slides off screen during playback.
- `elevationProfileOverlay` - slides up during playback; shows `ElevationProfileView` or a simple progress bar fallback.
- `ElevationProfileView` - renders a filled area chart of altitude vs. distance. Downsamples to max 200 points. Shows min/max altitude labels and a current-altitude callout.
- `StatCardView` - individual metric tile with icon, label, value, and unit. Supports a `spoiler` modifier that hides the value with an animated particle emitter (`CAEmitterLayer`) for locked metrics.
- `SpoilerView` / `SpoilerModifier` / `EmitterView` - `UIViewRepresentable` particle-emitter overlay used to obscure locked metric values.
- `WorkoutNavigationModel` (not in this file, but referenced) - manages nav state transitions.
- Playback timer - `Timer.publish(every: 1.0/60.0)` drives `playbackProgress` from 0 to 1 over `playbackDuration`.

## Data & Logic
- Route: fetched via `DataStore.shared.fetchRouteForWorkout(_:)`, then smoothed with `RouteSmoother.smooth(_:)` (Chaikin algorithm, 2 iterations). Coordinates and full `CLLocation` objects (for altitude) are stored.
- Altitude validation: requires at least 3 points with valid altitudes (between -1000 m and 10000 m) and at least 5 m total elevation range.
- Playback duration: < 1 km = 10 s, 1–5 km = 15 s, 5–10 km = 20 s, > 10 km = 30 s.
- Camera (normal mode): centers on route bounding box with 1.3× padding, shifts center slightly south so the route clears the bottom sheet.
- Camera (playback mode): centers on route bounding box with 1.5–1.7× padding for a wider view.
- Stats displayed:
  - Duration (always, free)
  - Distance (always, free)
  - Active Energy / Total Energy (Pro-locked)
  - Average Pace — hidden for Hiking (Pro-locked)
  - Elevation Gain — always shown for Hiking, shown for others only if > 0 (Pro-locked)
  - Average Heart Rate / Max Heart Rate — shown only if HealthKit data exists (Pro-locked)
- Weather: temperature and humidity shown in header if recorded at workout time; supports unit conversion (Celsius/Fahrenheit).
- Pace: derived from `averagePace` (minutes/km), converted to the user's distance unit.
- Elevation: displayed in meters only.

## Premium
- Route map is hidden for non-Pro users (blurred overlay + unlock prompt).
- Route playback (Play button) is Pro-only; tapping while non-Pro opens the paywall immediately.
- Active Energy, Total Energy, Average Pace, Elevation, Average Heart Rate, and Max Heart Rate are visually obscured with the spoiler effect for non-Pro users.
- Tapping "Unlock all metrics" or any Pro-gated element opens `PaywallView`.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Activity/ActivityDetailView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Utilities/RouteSmoother.swift`

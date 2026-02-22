# Activity Sharing

## Overview
A share sheet that lets users export a styled workout summary image or a GPX file. Triggered from the Activity Detail view via the Share button. Offers four visual templates, configurable map options, and a save-to-Photos action.

## User Experience
1. User taps the Share button (orange, in the Activity Detail toolbar).
2. `ActivitySharePreviewSheet` opens as a modal sheet.
3. A segmented picker at the top selects the share mode:
   - **Transparent** - stats card with optional inline route shape preview, transparent background (suitable for pasting on any background).
   - **Overlay** - map snapshot with stats overlaid in a gradient-fade panel at the bottom.
   - **Frame** - map snapshot framed with workout date above and stats below in monospaced text.
   - **Photo** - user picks a photo from their library; stats are overlaid on it exactly like Overlay mode.
   - Overlay and Frame modes are only shown when the workout has a recorded route (>= 2 coordinates).
4. A live image preview renders immediately. A `ProgressView` is shown while the image generates.
5. For Overlay, Frame, and Photo modes, additional pickers appear:
   - **Aspect Ratio**: 16:9, 4:3, 1:1 (default), 3:4, 9:16 — controls the map snapshot size.
   - **Map Type** (Overlay/Frame only): Standard or Satellite.
   - **Point of Interest** (Standard map only): Include All or Exclude All.
   - **Elevation Style** (Satellite only): Flat or Realistic.
6. For Photo mode, a "Select Photo" / "Change Photo" button opens `PhotosPicker`.
7. An **Appearance** picker (Light / Dark) is available for all non-Photo modes; it defaults to the inverse of the current system appearance.
8. The toolbar offers:
   - **Save to Photos** (orange download icon) - saves the current image as a PNG to the photo library (Pro-gated).
   - **Export GPX** (route icon, shown only when a route exists) - generates and shares a `.gpx` file via the system share sheet (Pro-gated).
9. Any action that requires Pro and is attempted by a non-Pro user opens `PaywallView`.

## Key Components
- `ActivitySharePreviewSheet` - main sheet view; owns all mode/configuration state and image generation.
- `ShareMode` enum - `.transparent`, `.overlay`, `.frame`, `.photo`; `useMapConfiguration` flag drives which sub-controls appear.
- `MapAspectRatio` enum - five ratios mapping to fixed pixel sizes (e.g., square = 600×600, 9:16 = 600×1067).
- `MapType` enum - `.standard` / `.imagery` (satellite).
- `POIFilter` enum - `.includeAll` / `.excludeAll`; maps to `MKPointOfInterestFilter`.
- `ElevationStyle` enum - `.flat` / `.realistic`; maps to `MKImageryMapConfiguration.elevationStyle`.
- `Appearance` enum - `.light` / `.dark`; applied to `UITraitCollection` for map snapshot rendering.
- `ActivityTransparentShareExportView` - renders the Transparent card: workout name, three stat rows (Distance, Pace/Elevation, Time), optional inline route shape drawn on a `Canvas`, and a "GetSteps.app" branding footer.
- `RoutePreviewView` (inside `ActivityTransparentShareExportView`) - a `Canvas`-based miniature route shape, latitude-corrected for aspect ratio.
- `MapOverlayShareExportView` - map image with a gradient-dark bottom overlay containing a 2-column grid of stats in white monospaced text.
- `MapFrameShareExportView` - map image with date header above and a single stats line below, both in monospaced text on the system background color.
- `generateMapWithRoute()` - uses `MKMapSnapshotter` to render a map at the selected aspect ratio, then draws the route polyline with `UIBezierPath` (6 pt, round cap/join, workout color).
- `renderTransparentImage()` - synchronous `ImageRenderer` call on the main thread.
- `renderOverlayImage()` / `renderFrameImage()` / `renderPhotoImage()` - async: map snapshot on background thread, then `ImageRenderer` on main thread.
- `cropImage(_:toAspectRatio:)` - aspect-fill crop for the user-selected photo to match the chosen ratio.
- `GPXExporter` - generates and saves the GPX file; see route-tracking.md.

## Data & Logic
- All four image variants are generated concurrently with `withTaskGroup` at `.userInitiated` priority when the sheet appears.
- Any configuration change (aspect ratio, map type, appearance, etc.) triggers a targeted regeneration of only the affected image variant.
- The share stat displayed as "Pace or Elevation" is chosen by caller: Hiking workouts receive elevation (in meters); all others receive pace (in min/unit).
- Map snapshot coordinates: bounding box of route coordinates with 1.2× padding on both axes.
- Trait collection is set explicitly on `MKMapSnapshotter.Options` so light/dark appearance is respected regardless of device theme.
- GPX export fetches raw route locations again from `DataStore` (unsmoothed), includes `<ele>` elements only when altitude >= 0.
- Save to Photos uses `PHAssetCreationRequest` with `UTType.png` and requests `.addOnly` authorization.

## Premium
- **Save to Photos** requires Pro; tapping while non-Pro opens `PaywallView`.
- **Export GPX** requires Pro; tapping while non-Pro opens `PaywallView`.
- The share preview image itself (all four modes) is visible to all users; only saving and GPX export are gated.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Activity/ActivitySharePreviewSheet.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Activity/MapShareExportView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Activity/WorkoutTransparentShareExportView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Utilities/GPXExporter.swift`

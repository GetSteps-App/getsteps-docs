# Route Tracking

## Overview
The system that records GPS coordinates during a workout, persists them across app restarts, smooths them for display, and exports them as GPX files. Powers the route polyline in both the live session map and the post-workout detail view.

## User Experience
- During a workout, the user's path is traced as a colored polyline on the live map in `SessionView`.
- After a workout, the smoothed route is displayed on the map in `ActivityDetailView` and used for route playback and share image generation.
- Users can export the route as a `.gpx` file from the share sheet (Pro-gated).
- Indoor workouts do not collect a full route — only the last known location is stored as a single point so HealthKit route association still works.

## Key Components

### LocationManager
- Singleton (`LocationManager.shared`), `CLLocationManagerDelegate`.
- `desiredAccuracy`: 5 metres for general location; used for weather fetching and current-position tracking.
- `requestLocationPermission()` - requests `whenInUse` authorization if undetermined; calls `requestCurrentLocation()` if already authorized.
- `requestCurrentLocation()` - single `locationManager.requestLocation()` call (non-continuous).
- `fetchWeatherData()` - fetches current weather via `WeatherKit.WeatherService` for `currentLocation`; stores a `WeatherData` struct in `Defaults[.currentWeatherData]` (temperature in Celsius, humidity as 0–100%).
- `refreshWeatherIfNeeded()` - respects a 15-minute refresh interval and a 5 km location-change threshold before re-fetching. Gated by `Defaults[.showWeather]` user preference.
- `fetchWeatherForWorkout()` - separate fetch path used during active workouts; respects a 5-minute interval independent of the general refresh.
- `shouldRefreshForLocationChange(_:)` - parses the stored location string, computes distance, returns `true` if > 5 km.
- Weather auto-refreshes on `didUpdateLocations` if the user has moved > 5 km from the last weather fetch location.

### WorkoutManager (route collection portion)
- `HKWorkoutRouteBuilder` accumulates GPS points and associates them with the finished `HKWorkout`.
- `CLLocationManager` (separate instance from `LocationManager.shared`) configured with:
  - `desiredAccuracy`: `kCLLocationAccuracyNearestTenMeters`
  - `activityType`: `.fitness`
  - `distanceFilter`: `kCLDistanceFilterNone` (all updates)
  - `pausesLocationUpdatesAutomatically`: `false`
- `shouldCollectFullRoute` - `true` when `locationType != .indoor`.
- `configureRouteCollectionIfNeeded(for:)` - lazily creates `routeBuilder` and the internal `CLLocationManager`; requests location authorization.
- `startRouteCollectionIfNeeded()` - enables background location updates (if `Always` authorization and `location` background mode declared), starts `locationManager.startUpdatingLocation()`.
- `stopRouteCollection()` / `teardownRouteCollection()` - stops updates, disables background mode, nils the route builder.
- `locationManager(_:didUpdateLocations:)` - filters to locations with `horizontalAccuracy` 0–50 m:
  - Outdoor: calls `persistRouteLocations(_:)` (saves to `Defaults[.workoutSessionLocations]`) and `routeBuilder.insertRouteData(_:)`.
  - Indoor: stores only `lastIndoorLocation` (most recent quality point).
- `persistRouteLocations(_:)` / `restorePersistedRouteLocationsIfNeeded()` - serialize/deserialize `WorkoutSessionLocation` structs (lat, lon, altitude, accuracies, course, speed, timestamp) to `UserDefaults` via the `Defaults` library so the route survives app termination.
- `finishRouteCollection(for:)` - called after the workout builder finishes; for outdoor, calls `routeBuilder.finishRoute(with:metadata:)`; for indoor, inserts the single last location first.
- Authorization escalation: starts with `whenInUse`, then requests `always` to enable background location.

### RouteSmoother
- `smooth(_: [CLLocation], iterations: Int = 2)` - smooths an array of `CLLocation` objects preserving altitude via Chaikin's corner-cutting algorithm.
- `smooth(_: [CLLocationCoordinate2D], iterations: Int = 2)` - coordinate-only overload (no altitude).
- Chaikin algorithm: for each pair of consecutive points, inserts two new points: Q = 0.75·P0 + 0.25·P1 and R = 0.25·P0 + 0.75·P1. First and last points are always preserved. Two iterations produce visually smooth curves without excessive point multiplication.
- Altitude is linearly interpolated using the same 0.75/0.25 weights, keeping elevation profiles consistent with the smoothed path.
- Applied in `ActivityDetailView.fetchRouteIfNeeded()` after loading raw locations from HealthKit.

### GPXExporter
- `TrackPoint` struct: `CLLocationCoordinate2D`, optional `elevation: Double`, `timestamp: Date`.
- `generateGPX(trackPoints:workoutName:workoutDate:)` - builds a GPX 1.1 XML string with `<metadata>`, `<trk>`, `<trkseg>`, and `<trkpt>` elements. Elevation is included as `<ele>` only when non-nil. Timestamps use ISO 8601 with fractional seconds (`withInternetDateTime | withFractionalSeconds`). XML special characters (`&`, `<`, `>`, `"`, `'`) are escaped.
- `saveToTemporaryFile(content:filename:)` - sanitizes the filename (replaces `/`, `:`, spaces), writes to `FileManager.default.temporaryDirectory` as a `.gpx` file, returns the `URL`.
- The file is shared via `UIActivityViewController` (system share sheet). No cleanup of temp files is performed by the app.

## Data & Logic
- Route data flow:
  1. `CLLocationManager` → quality filter (accuracy 0–50 m) → `Defaults[.workoutSessionLocations]` + `HKWorkoutRouteBuilder`.
  2. On workout end: `routeBuilder.finishRoute` associates route with `HKWorkout` in HealthKit.
  3. On detail view open: `DataStore.fetchRouteForWorkout` queries HealthKit for the `HKWorkoutRoute` → raw `CLLocation` array → `RouteSmoother.smooth` → stored as `routeCoordinates` + `routeLocations`.
  4. For sharing/GPX: raw (unsmoothed) locations re-fetched from `DataStore` to preserve original timestamps and exact coordinates.
- Session locations are persisted to `UserDefaults` (via Defaults library key `workoutSessionLocations`) on every batch of quality updates, enabling recovery after app kill.
- On `recoverWorkout`, persisted locations are replayed into the `routeBuilder` via `restorePersistedRouteLocationsIfNeeded()` before resuming collection.
- `clearPersistedRouteLocations()` is called at the start of a new workout and after reset to avoid stale data contaminating the next session.
- Weather data (`WeatherData`) is stored in `Defaults[.currentWeatherData]` and attached to the workout record at save time from the last fetched value.

## Premium
- Viewing the route in Activity Detail requires Pro (non-Pro users see a blurred map with an unlock prompt).
- GPX export from the share sheet requires Pro.
- Route collection during a live workout and route storage in HealthKit are available to all users.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Utilities/LocationManager.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Utilities/RouteSmoother.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Utilities/GPXExporter.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Workout/WorkoutManager.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Activity/ActivityDetailView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Activity/ActivitySharePreviewSheet.swift`

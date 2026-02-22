# Weather Display

## Overview
Optionally shows current weather conditions and temperature in the home screen header area. When enabled, the welcome/greeting message is replaced by a short weather description, an emoji icon, and the current temperature. Powered by Apple WeatherKit via a `LocationManager` that refreshes on app foreground. Includes a required Apple Weather legal attribution link.

## User Experience
1. When `showWeather` is enabled in settings and weather data is available, the header left side shows:
   - A short condition label (e.g., "Mostly Cloudy") in secondary style.
   - A weather emoji (e.g., "⛅") matching the condition.
   - The temperature in the user's preferred unit (Celsius or Fahrenheit) in tertiary style.
2. When weather is unavailable or disabled, the header shows the time-based greeting message instead (e.g., "Good morning").
3. Below the condition/temperature line, a small attribution line reads "Apple Weather Legal" with a tappable "Legal" link opening `https://weatherkit.apple.com/legal-attribution.html`. This is required by Apple's WeatherKit terms.
4. On iOS 26+ the weather content renders via `HeaderLeadingView` overlaid on the top-left of the `NavigationStack`. On earlier iOS it renders inside `HeaderView` in the `VStack` header.
5. Weather refreshes automatically each time the app becomes active via `locationManager.refreshWeatherIfNeeded()` called in `ContentView.onChange(of: scenePhase)`.

## Key Components
- `HeaderView` — pre-iOS 26 header. When `showWeather` is true and `currentWeatherData` is non-nil and has a valid `condition`, renders the weather row with `WeatherIconView` and temperature string; otherwise renders `welcomeMessage` text.
- `HeaderLeadingView` — iOS 26+ overlay variant. Same conditional logic; uses `.title3` font (slightly smaller than `HeaderView`'s `.title2`). Applies `.lineLimit(1).minimumScaleFactor(0.5)` and `.padding(.trailing, 120)` on the welcome message to avoid overlapping toolbar buttons.
- `WeatherIconView` — renders `condition.emoji` as a `Text` view. The emoji is resolved from a `WeatherCondition` extension on `WeatherKit.WeatherCondition`.
- `WeatherAttributionView` — an `HStack` with Apple logo (`applelogo` SF symbol), "Weather" text, and a `Link("Legal", ...)` pointing to the WeatherKit legal attribution URL. All items are `.secondary` colored at 10pt font size.
- `WeatherCondition` extension (`WeatherCondition.swift`) — adds `symbolName: String`, `color(for:) -> Color`, `emoji: String`, and `shortDescription: String` to all `WeatherKit.WeatherCondition` cases. Covers ~30 conditions.

## Data & Logic
- `currentWeatherData` is stored in `Defaults[.currentWeatherData]` as a `WeatherData` struct containing `temperature: Double` (Celsius) and `weatherCondition: WeatherCondition?`.
- `showWeather` is `Defaults[.showWeather]` (Bool, user-settable in Settings).
- `unitSystem` is `Defaults[.unitSystem]`. Temperature conversion: `unitSystem.convertedTemperature(fromCelsius:)` then `Int(rounded())` + `unitSystem.temperatureLabel` (e.g., "°C" or "°F").
- `LocationManager.refreshWeatherIfNeeded()` fetches the user's current location and calls WeatherKit; updates `Defaults[.currentWeatherData]` on success.
- `WeatherCondition.shortDescription` returns a localized string key (e.g., `"weather.mostlyCloudy"`) with an English default value.

## Premium
Not gated. Weather display is available to all users (requires location permission and device WeatherKit support).

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Home/HeaderView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Home/HeaderLeadingView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Home/WeatherIconView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Home/WeatherAttributionView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Models/WeatherCondition.swift`

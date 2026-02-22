# Paywall & Subscription

## Overview

The paywall is a full-screen cover that presents the Steps Pro subscription options. It is built on RevenueCat and offers three purchase tiers: a 7-day free trial (monthly billing), a yearly plan, and a lifetime one-time purchase. Individual and family plan variants are available. The paywall auto-rotates through 8 feature showcase cards before displaying pricing.

## User Experience

1. `PaywallView` is presented as a `fullScreenCover` from multiple entry points: the "Get Steps Pro" card in Settings, any Pro-gated feature tap, and the "View Plans" row for existing Pro users.
2. The screen has a black-to-orange-to-white vertical gradient background.
3. At the top, a horizontally paging `ScrollView` auto-advances every 5 seconds through 8 feature showcase cards:
   - Weekly & Monthly Chart
   - Workout Analytics
   - Watch Companion App
   - Home Screen Widgets
   - Goal Achievement Notifications
   - Route Maps & GPX Export
   - Color Themes
   - App Lock
4. Below the feature cards, three pricing rows are shown vertically:
   - **Pro Yearly** — shows yearly price, crossed-out monthly-equivalent price, and a "SAVE X%" chip. Green checkmark border when selected.
   - **7-Day Trial** — free trial, then monthly price. "FREE" badge in orange. Green checkmark border when selected.
   - **Lifetime** — one-time purchase price.
5. A "Continue" button (disabled and labelled "Current Plan" if the selected tier is already active) triggers purchase.
6. Below the button: "Switch to family plans" / "Switch to individual plans" toggle, "Restore Purchase" button, and Terms of Service / Privacy Policy links.
7. A "Redeem" button in the toolbar opens the App Store offer code redemption sheet.
8. During purchase or restore: a black 50% opacity overlay with a white spinner and "Processing..." label covers the screen.
9. On success: an alert shows "🎉 Purchase successful! Welcome to Steps Pro!" and dismisses the paywall.
10. On restore with no active subscription: alert shows the error. On failure: alert shows the error message.
11. The paywall records the current app version and current date to `Defaults[.lastPaywallVersionShown]` and `Defaults[.lastPaywallShownAt]` for 7-day cadence logic.

## Key Components

- `PaywallView` — root view; owns package state, purchase/restore logic, and alert presentation.
- `FeatureCards` — horizontal paging `ScrollView` with auto-advance timer; contains one view per Pro feature.
- Feature showcase views (one per Pro feature, all in `Paywall/`):
  - `WeeklyMonthlyChartPaywallView` — shows live mock weekly + monthly chart previews.
  - `AdvancedWorkoutAnalyticsPaywallView` — 2×2 grid of `StatCardView` tiles (calories, heart rate).
  - `AppleWatchCompanionPaywallView` — two watch screenshot images side by side.
  - `HomeScreenWidgetPaywallView` — offset step count widget and monthly calendar widget previews.
  - `NotificationPaywallView` — sample notification card with app icon, bold title, message body.
  - `RouteMapPaywallView` — live `Map` view with "Route Map" and "GPX Export" pill labels.
  - `ColorThemePaywallView` — row of 6 colour swatch pairs from `ColorThemePreset.presets`.
  - `AppLockPaywallView` — Face ID, shield, and Touch ID icons side by side.
- `ProChipView` — reusable tappable badge (e.g. "SAVE 30%", "CURRENT PLAN", "MOST POPULAR") that opens `PaywallView` when tapped; used both on the paywall and inline throughout the app.
- `PaywallLockView` — a blurred overlay with a "Pro" label placed over locked content areas; tapping opens `PaywallView` as a sheet.

## Data & Logic

- **RevenueCat** (`PurchaseManager`): loads offerings on appear via `purchaseManager.offerings`; individual offering ID vs family offering ID toggled by `showingFamilyPlan`.
- Package loading (`loadPackages()`): resolves `monthlyPackage` (`.monthly`), `yearlyPackage` (`.annual`), `lifetimePackage` (`.lifetime`) from the active offering; falls back to iterating `availablePackages` by `packageType`.
- Default selected package: `monthlyPackage` (7-day trial).
- Save percentage: `(monthlyPriceInYear - yearlyPrice) / monthlyPriceInYear`, shown as integer percent on the yearly chip.
- `purchase(package:)`: calls `purchaseManager.purchase(package:)` async; checks `isProUser` after to confirm success (handles edge cases where purchase completes but entitlement not immediately active).
- `restorePurchases()`: calls `purchaseManager.restorePurchases()` async; checks `isProUser` to determine success.
- `isCurrentPlan(_:)`: compares `package.storeProduct.productIdentifier` to `purchaseManager.activePlanProductId`.
- Offer code redemption: `AppStore.presentOfferCodeRedeemSheet(in:)` (StoreKit 2).
- iOS 26+: uses `NavigationView` with `scrollEdgeEffectStyle(.soft, for: .top)` toolbar; earlier iOS: variable-blur header overlay.

## Premium

The paywall screen itself is the entry point to Pro. Features gated behind Pro:

| Feature | Gate |
|---|---|
| Weekly & Monthly step chart view | `calendarViewMode` non-daily blocked |
| App Lock | Toggle redirects to paywall |
| Color Themes | Applied via `colorThemePreset` (gate at theme application layer) |
| Route Maps & GPX Export | `PaywallLockView` overlay on map |
| Home Screen Widgets (advanced) | Widget setting sheet gate |
| Goal Achievement Notifications | Notification paywall |
| Apple Watch Companion | Watch app entitlement |
| Advanced Workout Analytics | `PaywallLockView` on locked stat cards |

## Related Files

- `/Users/hieudinh/Projects/Steps/Steps/Paywall/PaywallView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Paywall/ProChipView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Paywall/PaywallLockView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Paywall/AppLockPaywallView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Paywall/ColorThemePaywallView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Paywall/WeeklyMonthlyChartPaywallView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Paywall/NotificationPaywallView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Paywall/RouteMapPaywallView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Paywall/HomeScreenWidgetPaywallView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Paywall/AppleWatchCompanionPaywallView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Paywall/AdvancedWorkoutAnalyticsPaywallView.swift`

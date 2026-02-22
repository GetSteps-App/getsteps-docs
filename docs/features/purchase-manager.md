# Purchase Manager

## Overview
`PurchaseManager` is the singleton that manages all in-app purchase and subscription state via the RevenueCat SDK. It tracks whether the user has an active "Pro" entitlement, exposes available offerings for the paywall UI, and handles purchase, restore, and real-time subscription update flows.

## User Experience
- On first launch, `configure()` is called at app startup. RevenueCat is initialized and offerings are fetched in the background.
- When the user opens the paywall, the UI reads `offerings` to display available packages (monthly, annual, lifetime) from the `"pro"` and `"pro-family"` offerings.
- Tapping a plan calls `purchase(package:)`. On success, `isProUser` flips to `true` and gated features unlock immediately.
- The "Restore Purchases" button calls `restorePurchases()`. If a prior purchase is found, `isProUser` is updated without a new charge.
- `isProUser` is persisted to `Defaults[.isProUser]` so the app knows the user's status on cold launch before the RevenueCat network call completes.
- `activePlanProductId` tracks which specific product (e.g., `"com.steps.monthly"`) is active, persisted to `Defaults[.activePlanProductId]`.
- `managementURL` is provided by RevenueCat for linking users to their subscription management page.

## Key Components
- **`PurchaseManager`** — `@Observable @MainActor` singleton. Properties:
  - `isProUser: Bool` — computed from `_isProUser`; in DEBUG builds, `debugOverrideProStatus` can override it.
  - `offerings: Offerings?` — RevenueCat offerings object; nil until `fetchOfferings()` completes.
  - `isLoadingOfferings: Bool` — true while fetching; used by paywall to show a loading state.
  - `activePlanProductId: String?` — identifier of the currently active subscription product.
  - `managementURL: URL?` — RevenueCat-provided subscription management URL.
- **`configure()`** — Calls `Purchases.configure(withAPIKey:)` with the RevenueCat public key. Subscribes to `customerInfoStream` for real-time updates and triggers `fetchOfferings()`.
- **`purchase(package:)`** — `async throws`. Calls `Purchases.shared.purchase(package:)`. On success, sets `_isProUser = true` and saves `activePlanProductId`. On failure or inactive entitlement, clears `activePlanProductId`.
- **`restorePurchases()`** — `async throws`. Calls `Purchases.shared.restorePurchases()` and routes result through `updateSubscriptionStatus(with:)`.
- **`fetchOfferings()`** — `async`. Sets `isLoadingOfferings = true`, fetches from RevenueCat, stores in `offerings`, then sets `isLoadingOfferings = false`.
- **`getCurrentOffering()`** — Returns `offerings?.current` (the default offering).
- **`getOffering(identifier:)`** — Returns a specific offering by ID (`"pro"` or `"pro-family"`).
- **`updateSubscriptionStatus(with:)`** — Private. Reads `customerInfo.entitlements["pro"]?.isActive`, updates `_isProUser`, `managementURL`, and `activePlanProductId`.
- **`setDebugProStatus(_:)`** — DEBUG only. Overrides `isProUser` without touching RevenueCat, useful for testing gated UI.

## Data & Logic
- RevenueCat API key: `"appl_oQMvAwPZIPWqBZZltHKriTymTOp"` (hardcoded in `configure()`).
- Pro entitlement key: `"pro"`.
- Individual offering ID: `"pro"`. Family offering ID: `"pro-family"`.
- `_isProUser` initializes from `Defaults[.isProUser]` at init, so the cached status is available synchronously on launch.
- `customerInfoStream` is an `AsyncStream` from RevenueCat; a `Task` runs for the app's lifetime consuming updates.
- Log level set to `.debug` in all builds (should be changed to `.error` for production if not already handled by build config).

## Premium
This class is the source of truth for Pro status. Features gated behind `isProUser` throughout the app include advanced analytics, historical data access beyond the free tier, additional widget options, and other premium capabilities defined elsewhere in the codebase.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Utilities/PurchaseManager.swift`

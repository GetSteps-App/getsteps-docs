# Yearly Receipt

## Overview

The Yearly Receipt is a decorative card at the top of the Insights screen styled to look like a physical paper receipt, complete with perforated edges and a dashed tear-line divider. It summarises the user's total steps, distance, and calories for the selected year, and displays a rotating "TOTAL" message instead of a monetary amount.

## User Experience

1. Appears as the first content card in the Insights scroll view, directly under the "Steps" section header.
2. The card has perforated top and bottom edges (scalloped shapes) and two internal perforated dividers separating the header, items, total, and footer.
3. Header: "👟 YOUR 2025 RECEIPT 👟" (or the selected year) in a monospaced headline font. The second shoe emoji is mirrored horizontally.
4. Items section: three rows — STEPS, DISTANCE, CALORIES — each with a caption label on the left and the value on the right. Values animate in with `contentTransition(.numericText())` on appear.
5. Total section: a rotating message instead of a price:
   - "PRICELESS 💎", "INVALUABLE ✨", "BEYOND MEASURE 📈", "OFF THE CHARTS 🔥", "10/10 🌟", "100% EARNED 💪", "PURE GOLD 🥇", "∞"
   - Selected by `totalSteps % 8`.
6. Footer: "THANK YOU FOR WALKING ⭐" in bold caption.
7. A subtle drop shadow (black 10% opacity, radius 4, y+2) gives the card slight lift.
8. Background adapts to colour scheme: near-white (`systemBackground`) in light mode, white at 5% opacity in dark mode.

## Key Components

- `YearlyReceiptView` — the receipt card itself.
- `PerforatedEdgeTop` / `PerforatedEdge` — custom `Shape` types that render the scalloped half-circle perforations along the top and bottom edges (defined in `PerforatedEdgeShapes.swift`).
- `TicketPerforatedDivider` — the notch cut-outs on the sides of the internal dashed divider line.
- `DashedLine` — a horizontal dashed line drawn via a `Shape` path, used for the internal tear lines.

## Data & Logic

- `steps`, `distance`, `calories` are formatted strings passed in from `InsightsView`:
  - Steps: decimal-formatted integer (`NumberFormatter` with `.decimal` style).
  - Distance: converted via `unitSystem.convertedDistance(fromKilometers:)` with 1 decimal place, plus unit label (km or mi).
  - Calories: formatted with 0 decimal places, suffixed "kcal".
- `messageIndex` is `yearlyTotalSteps`; modulo 8 selects the TOTAL message.
- `currentYear` reads `dataStore.selectedInsightYear` so the receipt header reflects whichever year is selected in the toolbar picker.
- The `didAppear` flag from `InsightsView` gates value display — before appearance, all values show "-" to allow numeric count-up animation on first render.

## Premium

This feature is **free**. No Pro subscription required.

## Related Files

- `/Users/hieudinh/Projects/Steps/Steps/Insights/YearlyReceiptView.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/PerforatedEdgeShapes.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Insights/InsightsView.swift`

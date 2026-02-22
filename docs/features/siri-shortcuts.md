# Siri Shortcuts

## Overview
The app exposes a single App Intent — "Get Today's Steps" — that integrates with Siri and the Shortcuts app. Users can trigger it by voice or build automations that branch on today's step count.

## User Experience
- After installing the app, Siri automatically learns the shortcut phrases and surfaces them as suggestions.
- User can say any of the registered phrases (e.g., "Hey Siri, get my steps from Steps") and Siri responds with the current step count as both a spoken dialog and a returned value.
- In the Shortcuts app, users can find the action under the Steps app and add it to any shortcut automation — for example: "If steps > 8000, send a message".
- The intent runs silently in the background without opening the app (`openAppWhenRun = false`).
- Siri speaks: "You have 7,432 steps today" (using the formatted integer).

## Key Components
- **`GetTodayStepsIntent`** — Conforms to `AppIntent`. Title: "Get Today's Steps". Description: "Returns your current step count for today". `openAppWhenRun = false`.
  - `perform()` reads today's record from `Defaults[.stepDailyRecords]`, extracts `stepCount`, returns it as `IntentResult & ReturnsValue<Int> & ProvidesDialog`.
  - Dialog string: `"You have \(formatted) steps today"`.
- **`StepsShortcuts`** — Conforms to `AppShortcutsProvider`. Registers one `AppShortcut` wrapping `GetTodayStepsIntent()` with 16 phrase variants covering natural language patterns.
  - Short title: "Today's Steps". System image: `"figure.walk"`.
  - Phrase variants include: "Get my steps from Steps", "How many steps in Steps", "How far have I walked in Steps", "How am I doing in Steps", etc.

## Data & Logic
- Data source: `Defaults[.stepDailyRecords]` — same shared App Group store used by widgets.
- Finds today's record via `Calendar.isDate(_:inSameDayAs:)`. Returns 0 if no record found.
- Step count formatted with `.number` style for the dialog string.
- The intent result value is a plain `Int` (the raw step count), usable as a variable in Shortcuts automations.

## Premium
No gating — Siri Shortcuts integration is available to all users.

## Related Files
- `/Users/hieudinh/Projects/Steps/Steps/Intents/GetTodayStepsIntent.swift`
- `/Users/hieudinh/Projects/Steps/Steps/Intents/StepsShortcuts.swift`

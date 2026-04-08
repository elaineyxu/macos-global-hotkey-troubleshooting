# Code Entrypoints

Inspect these in order.

## 1. Hotkey recording UI or browser capture path

Look for:

- settings panes or recorder components
- browser keyboard handlers
- native capture commands invoked from the frontend
- error handling around native capture

Questions:

- Can the browser path ever see `Fn`?
- Does the UI fall back to native capture?
- Are permission failures shown to the user or swallowed?

## 2. Config normalization and persistence path

Look for:

- fields like `hotkey`, `hotkey_mode`, `use_fn_hotkey`
- normalization helpers
- frontend store updates
- persistence calls that save to disk or backend config

Questions:

- Is native hotkey routing derived from the actual hotkey string or from sticky state?
- Does confirming a new hotkey truly persist it?
- Can the UI display a hotkey that the backend never registered?

## 3. Registration or sync path

Look for:

- startup registration
- update or sync functions
- code that unregisters and re-registers shortcuts
- branching between standard and native paths

Questions:

- When `use_fn_hotkey` is true, is standard registration intentionally skipped?
- When switching back to non-`Fn`, does standard registration resume?
- Are stale registrations left behind?

## 4. Native capture path

Look for:

- Objective-C or Swift capture helpers
- Rust or backend bridge code
- commands like capture, cancel, pause, resume, get-status
- events like `captured_hotkey`

Questions:

- Does capture mode prompt for permissions?
- What exact startup failure is reported?
- Is the callback or sender lifecycle safe if startup fails synchronously?

## 5. Native runtime monitor path

Look for:

- press and release events
- held-state booleans
- timers, debounce, watchdogs
- standalone `Fn` vs combo-key branches

Questions:

- What emits `hotkey_pressed`?
- What emits `hotkey_released`?
- Does combo release come from non-`Fn` key-up or a timer?
- Can stale `Fn` state leak into generic modifier logic?

## 6. Press or release to action or pipeline path

Look for:

- action dispatch from standard shortcut events
- native press or release to pipeline start or stop
- hold vs toggle branching
- direct click or manual invocation path

Questions:

- Is the hotkey path truly broken, or does the shared action fail after start?
- Can direct invocation reproduce the same failure?
- Are hotkey bugs being confused with downstream pipeline bugs?

## Interface concepts to identify explicitly

Capture these names in your notes:

- config fields: `hotkey`, `hotkey_mode`, `use_fn_hotkey`
- commands: capture, update, pause, resume, get-status
- events: `captured_hotkey`, `hotkey_pressed`, `hotkey_released`, status, error
- native reasons: `missing_input_monitoring`, `missing_accessibility`, `event_tap_create_failed`

If the repo uses different names, map them into the same conceptual buckets.

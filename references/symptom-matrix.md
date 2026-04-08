# Symptom Matrix

## `Fn` cannot be recorded

Likely subsystem:

- native capture path
- permission prompting
- UI error handling

Checks:

- does the browser path try to handle `Fn` directly
- does native capture start at all
- does capture mode prompt for Input Monitoring or Accessibility
- is a startup error emitted but hidden

Likely root causes:

- browser cannot observe `Fn`
- standard shortcut library used for capture
- capture mode permission prompt disabled
- capture error swallowed in frontend

## Recorded hotkey does not run

Likely subsystem:

- persistence
- config normalization
- registration or sync

Checks:

- does confirm persist to backend config
- does sync branch correctly between standard and native paths
- is the displayed hotkey different from the registered hotkey

Likely root causes:

- recorder updates frontend state only
- native-hotkey routing flag is sticky
- re-registration never happens after update

## Hold starts then instantly stops

Likely subsystem:

- native runtime monitor
- release timer
- held-state tracking

Checks:

- what emits the release event
- whether combo keys rely on timer-based release
- whether spurious `Fn` releases are expected on macOS

Likely root causes:

- release debounce window too aggressive or too broad
- combo target incorrectly released by `Fn` timer
- held-state boolean cleared too early

## Non-`Fn` hotkeys break after using `Fn`

Likely subsystem:

- config normalization
- routing between standard and native paths

Checks:

- whether `use_fn_hotkey` is derived from the current hotkey or sticky prior state
- whether standard registration is skipped when it should not be

Likely root causes:

- sticky native-hotkey flag
- fallback to standard global shortcut never restored

## Click path also broken

Likely subsystem:

- shared action or pipeline path

Checks:

- invoke the same action directly without hotkeys
- inspect start and stop failures after hotkey dispatch

Likely root causes:

- pipeline or backend start failure misread as hotkey bug
- missing API keys, audio setup, provider setup, or other downstream blockers

## Typing `Space` or normal input triggers false `Fn+Space`

Likely subsystem:

- held-state tracking
- stale modifier flags
- release debounce

Checks:

- whether `Fn` held state clears after true release
- whether `Fn` bit leaks into generic modifier tracking
- how long the monitor keeps `Fn` alive after release

Likely root causes:

- release window too wide
- stale `gFnHeldByUser`-style state
- generic release logic polluted by `Fn` flags

## Toggle mode behaves differently from hold mode

Likely subsystem:

- press or release to action path

Checks:

- does toggle path ignore release correctly
- does hold path start only from idle and stop only from recording

Likely root causes:

- release handlers not gated by mode
- shared action logic collapsing hold and toggle behavior

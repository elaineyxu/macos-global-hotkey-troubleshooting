# Permissions And System Behavior

## Core rule

Treat `Fn` or `Globe` as a macOS system-level key, not a normal browser-visible modifier.

Common consequences:

- Web and WebView keyboard events often cannot record `Fn` reliably.
- Standard global-shortcut libraries often cannot represent `Fn`.
- Native monitoring is usually required for `Fn` capture or runtime handling.

## Permissions to check first

Use this decision tree before patching code.

## Permission decision tree

1. Does the symptom only affect `Fn` or `Globe`, while normal shortcuts still work?
   Start by assuming the standard shortcut library is not enough. Check native interception and Input Monitoring first.
2. Does the app need to observe global key activity outside its own window?
   Check for Input Monitoring needs and native monitor startup failures.
3. Does the app also synthesize input, paste text, drive another app, or rely on trusted assistive APIs?
   Check Accessibility as well, even if the original symptom looks like a hotkey issue.
4. Does capture fail silently but runtime monitor works, or vice versa?
   Compare capture mode and monitor mode permission behavior separately.
5. Does direct click or direct action fail too?
   Do not over-attribute that to permissions for hotkey capture; it may be a downstream action path issue.

### Input Monitoring

Usually required when the app must observe global key activity via native interception.

Look for:

- explicit status APIs
- helper or monitor startup failures
- monitor works only inside the app but not globally
- capture or runtime monitor failing silently

### Accessibility

Often required when the app simulates input, intercepts certain global events, or depends on event-tap readiness.

Look for:

- `NSAccessibilityUsageDescription`
- explicit trust checks such as `AXIsProcessTrusted`
- errors mentioning Accessibility or assistive access

### Important nuance

Some repos need both permissions for `Fn`-based hotkeys:

- one to observe the key globally
- one to synthesize or complete the downstream action

Do not assume the same permission model applies to standard non-`Fn` hotkeys.

## Common permission interpretations

- `Fn` cannot be recorded, but normal keys can:
  likely native capture or Input Monitoring issue, or permission error hidden in UI
- `Fn` records, but runtime global behavior never fires:
  likely runtime monitor permission or monitor startup issue
- hotkey starts, but text insertion or simulated output fails:
  likely downstream Accessibility issue rather than capture failure
- both direct click and hotkey path fail:
  likely shared pipeline or action failure, not a permission-only hotkey bug

## Native interception mechanisms

### `CGEventTap`

Typical when the app needs low-level global key observation.

Good fit for:

- `Fn` / `Globe`
- `flagsChanged`
- modifier press or release state machines
- combo timing and debounce bugs

Common failure modes:

- event tap creation fails due to missing permission
- capture path and monitor path prompt permissions differently
- app receives partial events but not the ones needed for `Fn`

### `NSEvent` monitors

Can be used for higher-level event monitoring, but may not be sufficient for all `Fn` use cases.

Check whether the repo uses:

- local monitors
- global monitors
- only in-app event handling

If the symptom is global and `Fn`-specific, expect these to be insufficient unless paired with a lower-level native path.

### `flagsChanged`

This is commonly where modifier transitions are tracked.

Check for:

- special handling of keyCode `63` or equivalent `Fn` code
- held-state booleans
- timers or debounce windows
- branch differences between standalone `Fn` and `Fn+key`

## macOS quirks that matter

### Input-method or Globe-key switching

macOS may use the Globe or `Fn` key for input-method switching and can emit spurious press or release transitions.

Typical symptoms:

- hold starts then instantly stops
- real release is missed
- typing a normal key soon after `Fn` release looks like `Fn+key`

This matters more on newer macOS releases where Globe-key input-method behavior is more visible. Record the macOS version in your notes whenever the symptom involves `Fn`, `Globe`, or input-method switching.

## Environment edge cases

Record these if they might affect reproduction:

- external keyboards where `Fn` or Globe behavior differs from the built-in keyboard
- VMs or remote desktop sessions where event interception may be incomplete
- multiple input sources or rapid input-method switching
- secure text entry or app contexts where native interception behaves differently
- multi-monitor setups only if the app or helper is display-aware; otherwise treat them as lower priority than permission and state-machine checks

### Silent permission failures

A capture or monitor path may fail without surfacing a prompt if:

- prompting is disabled for that mode
- the helper emits an error but UI ignores it
- startup returns false and the frontend collapses into a generic failure

## What to record in your notes

- whether the repo uses `CGEventTap`, `NSEvent`, or both
- whether `Fn` is special-cased in native code
- which permission checks exist
- whether capture mode and monitor mode share the same permission behavior
- whether the downstream action requires separate Accessibility access

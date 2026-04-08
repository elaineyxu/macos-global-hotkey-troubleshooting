---
name: macos-global-hotkey-troubleshooting
description: Debug existing macOS desktop app codebases, especially Tauri, Electron, Swift, Objective-C, or Rust hybrids, where global shortcuts, Fn or Globe keys, native event interception, Accessibility, Input Monitoring, CGEventTap, NSEvent monitors, flagsChanged handlers, debounce and release bugs, or conflicts between standard global-shortcut libraries and native listeners are involved. Use when a macOS hotkey cannot be recorded, recorded hotkeys do not run, hold-to-record stops immediately, normal typing falsely triggers a hotkey, switching away from Fn breaks normal shortcuts, or a suspected hotkey bug may actually be a downstream pipeline or action failure.
---

# macOS Global Hotkey Troubleshooting

Use this skill for debugging existing macOS app codebases, not for broad product architecture discussion.

## Target platforms

This skill is written for macOS desktop apps with hybrid or native input stacks, especially:

- Tauri plus Rust plus Objective-C or Swift helpers
- Electron plus native macOS modules
- pure Swift or Objective-C macOS apps
- other desktop apps with one standard shortcut path and one native interception path

The workflow assumes there may be both:

- a standard library path for normal shortcuts
- a native path for `Fn`, `Globe`, or special interception cases

## When to use

Use this skill when the user asks to debug or understand:

- macOS global shortcut failures
- `Fn` / `Globe` hotkey capture or runtime issues
- Accessibility or Input Monitoring gating hotkeys
- `CGEventTap`, `NSEvent` monitor, `flagsChanged`, or native event interception problems
- conflicts between a standard global-shortcut library and a native listener
- hold or toggle behavior, false releases, debounce issues, or hotkey persistence bugs

Do not use this skill when the real issue is clearly downstream and unrelated to shortcut handling.

## Fast triage workflow

1. Confirm the symptom class before changing code.
   Distinguish recording, persistence, registration, runtime press or release, permissions, and shared pipeline failures.
2. Classify the active hotkey path.
   Decide whether the repo uses a standard shortcut library, a native monitor, or a hybrid split.
3. Check system prerequisites first.
   Confirm macOS-only behavior, required permissions, and whether permission failures can happen silently.
4. Inspect code entrypoints in a fixed order.
   Start at recording UI, then config normalization and persistence, then registration or sync, then native capture, then native runtime monitor, then press or release to action or pipeline.
5. Match the symptom to a known conflict pattern.
   Use the symptom matrix before inventing a new theory.
6. Validate with symptom-specific tests.
   Always test both the hotkey path and a direct action path if available.

## Quick walkthrough

Example symptom:

- User reports `Fn+Space` can be recorded in settings, but hold-to-record starts and stops immediately.

Recommended diagnosis sequence:

1. Confirm this is a runtime release problem, not a recording problem.
   The hotkey already records, so do not start in the recorder UI.
2. Classify the path as native or hybrid.
   Search for a native monitor, `CGEventTap`, `flagsChanged`, Objective-C or Swift helper, and Rust or backend bridge events like `hotkey_pressed` and `hotkey_released`.
3. Check whether the same downstream action works via direct click or command.
   If direct click also fails, stop and inspect the shared pipeline or action path instead of patching hotkey logic.
4. Inspect native runtime monitor code before editing.
   Look for debounce timers, held-modifier booleans, standalone `Fn` handling, combo-key release handling, and generic modifier-release logic.
5. Form the first concrete hypothesis.
   For this symptom, prefer “release is emitted too early for `Fn+key` combos” over “capture is broken”.
6. Verify the hypothesis with the smallest relevant test.
   Reproduce hold start, hold duration, and release behavior, then run the smallest backend build or typecheck for the touched surface.

## Evidence checklist

Collect these before proposing a fix:

- exact symptom and whether it is record-time or run-time
- macOS version if relevant
- active input method or Globe-key switching behavior if relevant
- standard shortcut library in use, if any
- native monitor implementation details, if any
- permission requirements and current app prompts or status APIs
- config fields controlling hotkey mode and native-vs-standard routing
- event names or callbacks used for capture, press, release, and status
- whether direct click or non-hotkey invocation also fails

## Symptom routing

- `Fn` cannot be recorded:
  inspect native capture and permission prompting first
- recorded hotkey does not run:
  inspect persistence, normalization, registration, and sync
- hold starts then instantly stops:
  inspect native press or release state machine and debounce
- normal typing triggers the hotkey:
  inspect held-modifier state, stale flags, and release timers
- switching away from `Fn` breaks normal hotkeys:
  inspect config routing and standard registration fallback
- hotkey and direct click both fail:
  inspect shared downstream action or pipeline before changing hotkey code

For detailed paths, read [references/symptom-matrix.md](./references/symptom-matrix.md).

## Required code inspection order

1. Hotkey recording UI or browser capture path
2. Config normalization and persistence path
3. Registration or sync path
4. Native capture path
5. Native runtime monitor path
6. Press or release to action or pipeline path

Use [references/code-entrypoints.md](./references/code-entrypoints.md) for what to look for at each step.

## Common conflict patterns

- browser or WebView cannot observe `Fn`
- standard shortcut library cannot represent `Fn`
- permission prompt exists in monitor mode but not capture mode
- permission error is emitted but swallowed in UI
- native-hotkey flag stays sticky and blocks standard shortcut registration
- recorder updates frontend state but does not persist backend config
- release timer emits too early for combo keys
- held-modifier state never clears
- `Fn` state leaks into generic modifier-release logic

Read [references/permissions-and-system-behavior.md](./references/permissions-and-system-behavior.md) and [references/symptom-matrix.md](./references/symptom-matrix.md) before patching native code.

## Verification checklist

Validate the exact reported symptom and nearby regressions:

- record `Fn`
- record `Fn+Space`
- switch from `Fn+Space` back to a normal hotkey
- verify hold-to-record start, hold, and stop
- verify toggle mode if supported
- run a false-trigger typing test
- test a direct action path to rule out shared backend failures
- run the smallest relevant build or typecheck for the changed surface

Use [references/verification.md](./references/verification.md) for the full matrix.

## References

- [references/permissions-and-system-behavior.md](./references/permissions-and-system-behavior.md)
- [references/code-entrypoints.md](./references/code-entrypoints.md)
- [references/symptom-matrix.md](./references/symptom-matrix.md)
- [references/verification.md](./references/verification.md)

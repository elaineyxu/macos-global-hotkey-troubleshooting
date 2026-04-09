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

## Verifying Accessibility permissions

Global hotkeys that rely on `CGEventTap`, input synthesis, or trusted assistive APIs require the Accessibility permission to be granted in macOS System Settings. A missing or revoked grant is a frequent root cause for silent hotkey failures. Always verify Accessibility status before debugging event-handling logic.

### When to check

- native capture or runtime monitor fails silently
- `CGEventTapCreate` returns `NULL`
- hotkey records but never fires at runtime
- downstream actions that synthesize input (paste, type, click) fail
- the app recently updated and macOS reset its trust entry

### Diagnostic steps

1. Open **System Settings → Privacy & Security → Accessibility**.
2. Confirm the app (or its helper binary) appears in the list and the toggle is enabled.
3. If the app was recently rebuilt or re-signed, remove it from the list and re-add it. macOS invalidates trust when the code signature changes.
4. Check whether the app uses a helper process. The helper must have its own Accessibility entry if it creates event taps independently.
5. After granting or revoking, relaunch the app. Some permission changes require a full restart to take effect.

### Programmatic verification

Use the examples below to check Accessibility trust status at runtime. Each example covers the query-only check and the prompt-based check that opens the system dialog.

#### Swift

```swift
import ApplicationServices

// Query-only check — returns current trust status without prompting
let trusted = AXIsProcessTrusted()
if !trusted {
    print("Accessibility permission not granted")
}

// Prompt-based check — shows the macOS system dialog if not already trusted
let options = [kAXTrustedCheckOptionPrompt.takeUnretainedValue(): true] as CFDictionary
let trustedWithPrompt = AXIsProcessTrustedWithOptions(options)
if !trustedWithPrompt {
    print("User has not yet granted Accessibility access")
}
```

#### Objective-C

```objc
#import <ApplicationServices/ApplicationServices.h>

// Query-only check
BOOL trusted = AXIsProcessTrusted();
if (!trusted) {
    NSLog(@"Accessibility permission not granted");
}

// Prompt-based check
NSDictionary *options = @{(__bridge NSString *)kAXTrustedCheckOptionPrompt: @YES};
BOOL trustedWithPrompt = AXIsProcessTrustedWithOptions((__bridge CFDictionaryRef)options);
if (!trustedWithPrompt) {
    NSLog(@"User has not yet granted Accessibility access");
}
```

#### Tauri (Rust backend with JavaScript frontend)

Tauri apps run native code in Rust. Use the `core-foundation` and `core-graphics` crates or call the macOS C API directly via FFI. Expose the result to the JavaScript frontend through a Tauri command.

Rust side (`src-tauri/src/lib.rs` or a dedicated module):

```rust
use std::ptr;

#[link(name = "ApplicationServices", kind = "framework")]
extern "C" {
    fn AXIsProcessTrusted() -> bool;
    fn AXIsProcessTrustedWithOptions(options: *const std::ffi::c_void) -> bool;
}

#[tauri::command]
fn check_accessibility_trusted() -> bool {
    // Safety: AXIsProcessTrusted is a stable macOS C API with no parameters
    unsafe { AXIsProcessTrusted() }
}

#[tauri::command]
fn request_accessibility_prompt() -> bool {
    // Safety: constructs a CFDictionary with kAXTrustedCheckOptionPrompt = true
    // and passes it to AXIsProcessTrustedWithOptions
    unsafe {
        use core_foundation::base::TCFType;
        use core_foundation::boolean::CFBoolean;
        use core_foundation::dictionary::CFDictionary;
        use core_foundation::string::CFString;

        let key = CFString::new("AXTrustedCheckOptionPrompt");
        let value = CFBoolean::true_value();
        let options = CFDictionary::from_CFType_pairs(&[(key, value)]);
        AXIsProcessTrustedWithOptions(options.as_concrete_TypeRef() as *const _)
    }
}
```

JavaScript side:

```javascript
import { invoke } from "@tauri-apps/api/core";

async function checkAccessibility() {
  const trusted = await invoke("check_accessibility_trusted");
  if (!trusted) {
    console.warn("Accessibility permission not granted");
    // Optionally prompt the user via the system dialog
    const prompted = await invoke("request_accessibility_prompt");
    if (!prompted) {
      console.error("User has not yet granted Accessibility access");
    }
  }
  return trusted;
}
```

#### Electron

Electron provides a built-in API on macOS for checking and requesting Accessibility trust.

Main process (`main.js` or equivalent):

```javascript
const { systemPreferences } = require("electron");

// Query-only check — does not prompt
const trusted = systemPreferences.isTrustedAccessibilityClient(false);
if (!trusted) {
  console.warn("Accessibility permission not granted");
}

// Prompt-based check — opens the macOS system dialog if not trusted
const trustedWithPrompt = systemPreferences.isTrustedAccessibilityClient(true);
if (!trustedWithPrompt) {
  console.error("User has not yet granted Accessibility access");
}
```

Expose to the renderer process via IPC if the check is needed from the frontend:

```javascript
// main process — preload or ipcMain handler
const { ipcMain, systemPreferences } = require("electron");

ipcMain.handle("check-accessibility", (_event, prompt) => {
  return systemPreferences.isTrustedAccessibilityClient(prompt ?? false);
});

// renderer process
const { ipcRenderer } = require("electron");

async function checkAccessibility(prompt = false) {
  const trusted = await ipcRenderer.invoke("check-accessibility", prompt);
  if (!trusted) {
    console.warn("Accessibility permission not granted");
  }
  return trusted;
}
```

### What to look for in the codebase

When auditing an existing project for correct Accessibility handling:

- search for `AXIsProcessTrusted` and `AXIsProcessTrustedWithOptions` calls
- confirm the check runs before `CGEventTapCreate` or native monitor startup
- verify the result is surfaced to the user, not silently swallowed
- check whether the prompt option is used at an appropriate time, such as first launch or settings entry, rather than on every keystroke
- look for `NSAccessibilityUsageDescription` in `Info.plist` — macOS may require it for the prompt dialog
- confirm helper binaries have their own trust checks if they create event taps independently

### Common mistakes

- checking trust once at launch but never rechecking after the user grants it, requiring a restart
- calling `AXIsProcessTrustedWithOptions` with prompt on every event-tap retry, spamming the dialog
- assuming Accessibility alone is sufficient when Input Monitoring is also required for global observation
- not handling the code-signature invalidation case where a rebuild revokes the prior trust entry
- testing only in development builds where Accessibility is already granted and missing the first-launch experience

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

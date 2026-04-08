# Verification

## Core principle

Do not declare a hotkey fix complete until you verify:

- the reported symptom is gone
- adjacent hotkey modes still work
- the direct non-hotkey action path still works

Always write down which checks were manual and which were automated.

## Permission-state tests

Test both granted and denied states when possible:

- try native capture with missing Input Monitoring
- try native capture with missing Accessibility if relevant
- confirm the app surfaces a concrete reason instead of failing silently
- confirm permission status APIs and UI messages agree

Manual procedure:

1. Revoke the relevant permission in macOS System Settings if safe to do so in your environment.
2. Relaunch the app if the repo or platform requires restart for permission refresh.
3. Re-run the exact failing capture or runtime action.
4. Record the exact user-visible message and any emitted internal reason.

## Recording tests

- record standalone `Fn`
- record `Fn+Space`
- record a standard non-`Fn` hotkey
- switch from `Fn+Space` back to a standard hotkey

Expected outcomes:

- recorded value persists
- active registration matches the displayed hotkey
- native routing is only used when required

Manual procedure:

1. Open the settings or recorder UI.
2. Record the target hotkey.
3. Save or confirm it.
4. Re-open settings or reload the app if needed.
5. Confirm the stored hotkey still matches what was recorded.

## Runtime behavior tests

### Hold mode

- press and hold the hotkey
- confirm action starts once
- confirm it remains active while held
- confirm it stops on the correct release event

Manual procedure:

1. Trigger the hotkey once.
2. Watch for start state.
3. Keep holding long enough to prove it does not instantly stop.
4. Release the intended key and confirm stop occurs once.

### Toggle mode

- press once to start
- press again to stop
- confirm release alone does not stop toggle mode unintentionally

Manual procedure:

1. Trigger the hotkey once and confirm start.
2. Release all keys and confirm the app keeps running in toggle mode.
3. Trigger the same hotkey again and confirm stop.

## False-trigger tests

- type normally after releasing `Fn`
- test `Space` soon after `Fn` release
- switch input methods if Globe-key switching is relevant

Expected outcome:

- normal typing should not trigger the hotkey
- false release and false combo behavior should not recur

Manual procedure:

1. Release `Fn`.
2. Immediately type `Space` or the key that previously caused false triggers.
3. Repeat several times, including while switching input methods if relevant.
4. Confirm no hotkey action is fired.

## Regression tests

Always test:

- `Fn` capture
- `Fn` runtime behavior
- normal non-`Fn` hotkeys
- direct click or direct action invocation

If click or direct action fails too, treat that as a likely downstream failure, not a hotkey-only regression.

## Build and static verification

Run the smallest relevant validation for the changed surface:

- native or backend build check
- frontend typecheck or build check
- targeted tests if they exist for hotkey logic

Command templates:

```bash
cargo check --manifest-path src-tauri/Cargo.toml
```

```bash
npm run build
```

```bash
npx tsc --noEmit
```

```bash
rg -n "hotkey|shortcut|fn|globe|event tap|flagsChanged" .
```

Use the smallest subset that matches the repo:

- Tauri or Rust backend: `cargo check`
- web or frontend changes: `npx tsc --noEmit` or `npm run build`
- codebase orientation pass: `rg`

If the repo already has targeted tests, prefer those over broad end-to-end runs.

## Dry-run scenarios this skill should handle

Use these as self-check examples:

- `Fn` cannot be recorded in settings
- `Fn+Space` starts and instantly stops
- typing space accidentally triggers `Fn+Space`
- switching away from `Fn` breaks normal shortcuts
- hotkey and direct click both fail

If your diagnosis path would send any of these to the wrong subsystem first, tighten the investigation before patching.

# macos-global-hotkey-troubleshooting

A Claude Code skill for debugging global hotkey failures in macOS desktop apps.

## What this skill does

When invoked, this skill gives Claude a structured diagnostic workflow for hotkey bugs — covering recording failures, runtime press/release issues, permission problems, and false triggers. It works on hybrid stacks (Tauri, Electron, native Swift/Objective-C, Rust) where a standard shortcut library and a native interception path coexist.

## When to use it

- `Fn` or `Globe` key cannot be recorded
- A recorded hotkey does not fire at runtime
- Hold-to-record starts then instantly stops
- Normal typing accidentally triggers the hotkey
- Switching away from `Fn` breaks standard shortcuts
- Hotkey and direct-click both fail (shared pipeline bug)

## Installation

```bash
claude skills install https://github.com/elaineyxu/macos-global-hotkey-troubleshooting
```

## What's included

| File | Purpose |
|---|---|
| `SKILL.md` | Main skill definition, triage workflow, and symptom routing |
| `references/symptom-matrix.md` | Symptom → subsystem → root cause mapping |
| `references/permissions-and-system-behavior.md` | Input Monitoring, Accessibility, CGEventTap, macOS quirks |
| `references/code-entrypoints.md` | Ordered inspection guide for 6 code surfaces |
| `references/verification.md` | Manual test procedures and build verification commands |

## Target platforms

- Tauri + Rust + Objective-C or Swift helpers
- Electron + native macOS modules
- Pure Swift or Objective-C macOS apps
- Other apps with a standard shortcut path and a native interception path

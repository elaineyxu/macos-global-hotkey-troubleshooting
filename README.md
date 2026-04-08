# macos-global-hotkey-troubleshooting

A structured debugging guide for global hotkey failures in macOS desktop apps.

Covers recording failures, runtime press/release bugs, permission problems, and false triggers. Works on hybrid stacks (Tauri, Electron, native Swift/Objective-C, Rust) where a standard shortcut library and a native interception path coexist.

## When to use

- `Fn` or `Globe` key cannot be recorded
- A recorded hotkey does not fire at runtime
- Hold-to-record starts then instantly stops
- Normal typing accidentally triggers the hotkey
- Switching away from `Fn` breaks standard shortcuts
- Hotkey and direct-click both fail (shared pipeline bug)

## What's included

| File | Purpose |
|---|---|
| `SKILL.md` | Triage workflow and symptom routing |
| `references/symptom-matrix.md` | Symptom → subsystem → root cause mapping |
| `references/permissions-and-system-behavior.md` | Input Monitoring, Accessibility, CGEventTap, macOS quirks |
| `references/code-entrypoints.md` | Ordered inspection guide for 6 code surfaces |
| `references/verification.md` | Manual test procedures and build verification commands |

## Target platforms

- Tauri + Rust + Objective-C or Swift helpers
- Electron + native macOS modules
- Pure Swift or Objective-C macOS apps
- Other apps with a standard shortcut path and a native interception path

## Usage

**Read directly** — the reference files are self-contained and can be shared with any AI coding agent or used as a manual debugging checklist.

**Claude Code** — install as a skill so Claude can invoke the workflow automatically:

```bash
claude skills install https://github.com/elaineyxu/macos-global-hotkey-troubleshooting
```

**OpenAI Codex** — clone the repo and reference the files in your session:

```bash
git clone https://github.com/elaineyxu/macos-global-hotkey-troubleshooting
codex "Debug this hotkey issue" --context macos-global-hotkey-troubleshooting/SKILL.md
```

**Cursor** — add the files to your project rules so Cursor picks them up automatically:

```bash
git clone https://github.com/elaineyxu/macos-global-hotkey-troubleshooting .cursor/rules/macos-global-hotkey-troubleshooting
```

Then reference the guide in your Cursor chat: `use @macos-global-hotkey-troubleshooting/SKILL.md`.

## Credits

Written by [OpenAI Codex](https://openai.com/codex).

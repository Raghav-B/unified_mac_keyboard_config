# macOS keyboard compatibility layer

This repository makes a Windows/Linux-style keyboard workflow feel natural on macOS, using [Karabiner-Elements](https://karabiner-elements.pqrs.org/) plus VS Code keybindings.

## What it changes

- In normal macOS applications, physical `Ctrl` behaves as `Command`, so `Ctrl+C`, `Ctrl+V`, `Ctrl+Z`, and similar familiar shortcuts work.
- In standalone terminals, real Unix `Ctrl` behavior is kept intact (`Ctrl+C`, `Ctrl+D`, `Ctrl+A`, and so on). Terminal copy/paste/new-tab shortcuts use `Ctrl+Shift`.
- VS Code is handled separately so its editor gets Windows/Linux shortcuts while its integrated terminal still receives real `Ctrl` input.
- Recreates a few system shortcuts: `Alt+Tab` for app switching, tap `Win` for Mission Control, `Shift+Win+S` for clipboard screenshots, and `Ctrl+Alt+T` to open Terminal.
- Makes `Home` and `End` act like Windows/Linux line-navigation keys, with terminal-specific behavior for shell editing.

## Files

- `karabiner.json` — Karabiner-Elements profile and system-wide mappings.
- `vscode_keybindings.json` — VS Code bindings, including editor-versus-integrated-terminal handling.

## Install/update

Install Karabiner-Elements, then use `karabiner.json` as the active configuration at `~/.config/karabiner/karabiner.json`. Copy the contents of `vscode_keybindings.json` into VS Code’s `keybindings.json`.

The design is intentionally opinionated: prioritize Windows/Linux muscle memory without sacrificing native Unix terminal controls.

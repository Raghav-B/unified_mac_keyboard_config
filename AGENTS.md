# AGENTS.md

## Purpose

This repository contains a macOS keyboard/input compatibility configuration intended to make macOS behave more like Linux/Windows while preserving native Unix terminal semantics.

The primary design goal is:

> Avoid having to maintain separate keyboard muscle memory when switching between macOS, Linux, and Windows.

The configuration is intentionally opinionated. Do not "simplify" mappings back to conventional macOS behavior unless explicitly requested.

## Repository Structure

```text
.
├── .gitignore
├── AGENTS.md
├── karabiner.json
└── vscode_keybindings.json
```

### `karabiner.json`

Karabiner-Elements configuration containing global and application-specific keyboard remappings.

The live macOS Karabiner configuration normally resides at:

```text
~/.config/karabiner/karabiner.json
```

The repository version is the source-controlled copy.

### `vscode_keybindings.json`

VS Code-specific keybindings used to reproduce Windows/Linux-style shortcuts inside VS Code while allowing its integrated terminal to retain real Unix `Ctrl` semantics.

This is necessary because Karabiner can detect that VS Code is the frontmost application, but cannot determine whether keyboard focus is currently inside VS Code's integrated terminal.

VS Code's own `when` clauses such as `terminalFocus` should therefore be used for this distinction.

---

# Desired Keyboard Model

The conceptual model is:

## Normal macOS Applications

Physical `Ctrl` should behave as macOS `Command`.

Physical `Command` / Windows key should generally behave as macOS `Control`.

Examples:

```text
Ctrl+C       → Copy
Ctrl+V       → Paste
Ctrl+X       → Cut
Ctrl+Z       → Undo
Ctrl+A       → Select All
Ctrl+T       → New Tab
Ctrl+W       → Close
```

The goal is Windows/Linux muscle memory.

## Standalone Terminals

Terminal applications are deliberately excluded from the global Ctrl/Command swap.

Known terminal bundle identifiers include:

```text
com.apple.Terminal
com.googlecode.iterm2
dev.warp.Warp-Stable
com.mitchellh.ghostty
```

Inside these applications:

```text
Physical Ctrl → actual Control
Physical Cmd  → actual Command
```

Therefore normal Unix terminal behavior remains intact:

```text
Ctrl+C        → SIGINT
Ctrl+D        → EOF
Ctrl+Z        → suspend
Ctrl+A        → shell beginning-of-line
Ctrl+E        → shell end-of-line
```

Linux-style terminal shortcuts are layered on top:

```text
Ctrl+Shift+C  → Copy
Ctrl+Shift+V  → Paste
Ctrl+Shift+T  → New tab
Ctrl+Shift+W  → Close tab/window
```

Do not globally convert Ctrl into Command inside terminal applications.

---

# VS Code Special Case

VS Code is excluded from the Karabiner Ctrl/Command swap:

```text
com.microsoft.VSCode
```

This is intentional.

Karabiner only sees VS Code as the frontmost application. It cannot distinguish:

```text
VS Code editor
```

from:

```text
VS Code integrated terminal
```

Therefore VS Code behavior should be implemented in `vscode_keybindings.json`.

The intended behavior is:

### Editor

```text
Ctrl+C → Copy
Ctrl+V → Paste
Ctrl+X → Cut
Ctrl+Z → Undo
Ctrl+A → Select All
Ctrl+S → Save
Ctrl+F → Find
Ctrl+P → Quick Open
...
```

### Integrated Terminal

```text
Ctrl+C        → SIGINT
Ctrl+D        → EOF
Ctrl+Z        → suspend
Ctrl+Shift+C  → Copy
Ctrl+Shift+V  → Paste
```

Prefer VS Code `when` clauses such as:

```text
terminalFocus
!terminalFocus
```

when implementing behavior that differs between the editor and terminal.

Do not attempt to solve integrated-terminal focus detection using Karabiner bundle identifiers.

---

# System Shortcuts

Several Windows/Linux-style system shortcuts are deliberately recreated.

## Application Switching

```text
Alt+Tab → Cmd+Tab
```

Both left and right Option/Alt variants may be supported.

The intended physical interaction is the familiar Windows/Linux `Alt+Tab`.

## Spotlight

The configuration may expose Spotlight through combinations such as:

```text
Win+Space
```

Terminal-specific mappings may also expose:

```text
Ctrl+Space → Spotlight
```

when a standalone terminal is focused.

## Mission Control

The physical Windows key is intended to act as a system-level key when tapped.

Desired behavior:

```text
Tap Win → Mission Control
```

Mission Control can be invoked using the equivalent macOS shortcut:

```text
Ctrl+Up
```

Be careful when modifying this rule because the physical Windows key is reported as `left_command`, which also participates in the global Command→Control swap.

A `left_command` manipulator implementing `to_if_alone` can consume the key before a later Command→Control manipulator sees it.

If the tap behavior and held behavior are implemented by the same manipulator, the intended conceptual behavior is:

```text
Tap Win  → Mission Control
Hold Win → Control
```

Avoid multiple competing manipulators that independently consume `left_command`.

## Screenshot

Desired Windows-style screenshot behavior:

```text
Shift+Win+S
```

should start rectangular screenshot selection and copy the result directly to the clipboard.

The corresponding macOS shortcut is:

```text
Cmd+Ctrl+Shift+4
```

## Terminal Launcher

Desired physical shortcut:

```text
Ctrl+Alt+T → open a new macOS Terminal window
```

This should work regardless of whether focus is currently in:

* a normal macOS application
* Terminal.app
* VS Code
* the VS Code integrated terminal

Because Ctrl is swapped to Command in normal applications but remains actual Ctrl in Terminal and VS Code, multiple Karabiner mappings may be required to cover both the pre-swap and post-swap modifier states.

Do not remove apparently duplicate Terminal-launch mappings without checking this distinction.

Terminal is currently launched using a command equivalent to:

```sh
open -na /System/Applications/Utilities/Terminal.app
```

The `-n` behavior is intentional: the shortcut should create a new Terminal instance/window even if Terminal is already running.

---

# Home / End Behavior

macOS's default Home/End behavior is undesirable for this configuration.

The desired Windows/Linux behavior is:

```text
Home        → beginning of line
End         → end of line
Shift+Home  → select to beginning of line
Shift+End   → select to end of line
```

For ordinary macOS text fields, this can be approximated using:

```text
Home → Cmd+Left
End  → Cmd+Right
```

and:

```text
Shift+Home → Cmd+Shift+Left
Shift+End  → Cmd+Shift+Right
```

Standalone terminals are special.

The current configuration may translate:

```text
Home → Ctrl+A
End  → Ctrl+E
```

for shell line editing.

Be aware that this is not semantically identical to sending actual Home/End keys.

Programs such as:

```text
vim
nano
less
fzf
tmux
ssh
```

may interpret Ctrl+A/Ctrl+E differently.

If changing this behavior, test both normal zsh command-line editing and interactive terminal applications.

---

# Terminal `exit` Behavior

The desired Terminal.app behavior is:

```text
If multiple Terminal sessions/windows exist:
    exit → close only the current shell/window

    If this is the final Terminal session/window:
        exit → quit Terminal.app entirely
        ```

        This behavior is implemented at the shell/macOS level rather than purely through Karabiner.

        Do not assume Karabiner is responsible for `exit` behavior.

        ---

# Karabiner Rule Ordering

        Ordering is important.

        Karabiner complex modifications are not a general cascading rewrite system. A broad modifier mapping can consume an input event before a later, more-specific mapping sees it.

        As a general principle:

        ```text
        specific shortcut exceptions
                ↓
                application-specific behavior
                        ↓
                        tap/hold modifier behavior
                                ↓
                                broad modifier swaps
                                ```

                                Broad mappings such as:

                                ```text
                                Ctrl → Command
                                Command → Control
                                ```

                                should generally appear after more specific shortcuts such as:

                                ```text
                                Ctrl+Alt+T
                                Shift+Win+S
                                Win+Space
                                Alt+Tab
                                ```

                                However, modifier key-down events require particular care. Merely putting a later `Ctrl+Something` rule before or after a modifier-remapping rule does not always produce intuitive chaining behavior.

                                When uncertain, use Karabiner-EventViewer to inspect what is actually emitted.

                                ---

# Important Karabiner Pitfalls

## First Matching Manipulator

                                Avoid multiple enabled manipulators that consume the same modifier event unless their conditions make them mutually exclusive.

                                For example, two unconditional rules beginning with:

                                ```json
                                "from": {
                                        "key_code": "left_command"
                                }
```

are likely to conflict.

This is especially important for the Windows-key/Mission-Control behavior and the global Command→Control swap.

## `optional: ["any"]`

Use this intentionally.

For example:

```json
"optional": ["any"]
```

means additional modifiers are allowed to participate in the match.

This is convenient for some shortcuts but can cause unexpectedly broad matches.

Prefer exact modifier requirements where exact semantics matter.

## Physical vs Logical Modifiers

Always distinguish between:

1. the physical key being pressed
2. the HID key Karabiner receives
3. the modifier emitted after remapping
4. the shortcut macOS/application ultimately sees

For the external Windows-style keyboard:

```text
Physical Windows key ≈ Command key to macOS
Physical Alt key     ≈ Option key to macOS
Physical Ctrl key    ≈ Control key to macOS
```

The global remapping then changes some of those meanings.

When debugging a shortcut, reason through the complete transformation chain.

## Terminal Detection

Karabiner's terminal exclusions are based on the frontmost application's bundle identifier.

This works for standalone terminal applications.

It does NOT detect:

* VS Code integrated terminal focus
* terminals embedded inside other IDEs
* terminal widgets embedded in arbitrary applications

Application-level keybindings should be used where finer context is required.

---

# Testing Changes

After modifying keyboard behavior, test at minimum:

## Normal Application

Use a browser or ordinary text editor and verify:

```text
Ctrl+C
Ctrl+V
Ctrl+A
Ctrl+Z
Ctrl+T
Ctrl+W
```

## Terminal.app

Verify:

```text
Ctrl+C
Ctrl+D
Ctrl+A
Ctrl+E
Ctrl+Shift+C
Ctrl+Shift+V
Home
End
```

Do not test `Ctrl+D` against an important live shell unless exiting it is acceptable.

## VS Code Editor

Verify normal Windows/Linux editing shortcuts.

## VS Code Integrated Terminal

Verify:

```text
Ctrl+C
Ctrl+Shift+C
Ctrl+Shift+V
```

and ensure Ctrl+C reaches the terminal rather than becoming Copy.

## System Shortcuts

Verify:

```text
Alt+Tab
Win+Space
Tap Win
Shift+Win+S
Ctrl+Alt+T
```

Test `Ctrl+Alt+T` from both:

* a normal application
* Terminal.app
* VS Code

because modifier behavior differs between those contexts.

---

# Debugging

Use Karabiner-EventViewer whenever observed behavior differs from expectations.

Do not guess what a physical key is producing.

Inspect:

```text
key_code
modifier state
key_down
key_up
```

and reason from the actual events.

For application-specific rules, also verify the frontmost application's bundle identifier.

---

# Change Philosophy

When making changes:

1. Preserve Windows/Linux muscle memory wherever practical.
2. Preserve genuine Unix Ctrl semantics inside terminals.
3. Prefer small, logically separated Karabiner rules over one enormous rule.
4. Use VS Code keybindings for VS Code terminal/editor context differences.
5. Keep special-case shortcuts ahead of broad modifier remaps where appropriate.
6. Avoid duplicate manipulators competing for the same physical modifier.
7. Test normal apps, Terminal.app, and VS Code after modifier-level changes.
8. Do not replace working behavior merely to make the JSON more aesthetically simple.
9. Keep `karabiner.json` and `vscode_keybindings.json` under version control.
10. Commit known-good states before significant modifier-remapping experiments.

The configuration is intentionally a compatibility layer over macOS rather than an attempt to follow standard macOS keyboard conventions.


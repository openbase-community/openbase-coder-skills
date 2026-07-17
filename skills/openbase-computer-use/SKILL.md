---
name: openbase-computer-use
description: >-
  Use this skill when an agent needs to operate the user's visible desktop
  through Computer Use tools. Routes to the correct path based on agent
  environment and operating system.
version: 0.3.0
---

# Openbase Computer Use

Computer use is intentionally visible. The user can stop the operation at any
time. Treat any user request to stop, pause, abort, or interrupt as an immediate
instruction to stop issuing further desktop actions.

## Which path to follow

Determine your environment and follow the corresponding sub-file:

| Environment | File |
|---|---|
| Codex agent running on macOS | [codex-macos.md](codex-macos.md) |
| Claude Code running on macOS | [claude-code-macos.md](claude-code-macos.md) |
| Any agent running on Linux | [linux.md](linux.md) |

**How to tell which environment you are in:**

- If you have `mcp__computer-use__*` tools available (e.g. `request_access`,
  `screenshot`, `left_click`) and you are Claude Code → use
  [claude-code-macos.md](claude-code-macos.md).
- If the host OS is Linux → use [linux.md](linux.md) regardless of agent type.
- If you are a Codex agent on macOS → use [codex-macos.md](codex-macos.md).

## Manual iOS Remote Control Screen Share

Use this when the user wants to control a desktop manually from the Openbase iOS
app. The command depends on the host OS.

On macOS (drives the Electron desktop app's companion):

```
openbase-coder desktop screen-share start
openbase-coder desktop screen-share stop
```

On Linux (drives the DevSpace desktop's Linux companion; the `desktop
screen-share` command is macOS-only and will refuse to run):

```
openbase-coder computer-use screen-share start
openbase-coder computer-use screen-share stop
```

After the screen-share tile appears in iOS, the user opens it full screen and
taps Remote to enable manual control.

This is not an AI computer-use run. Do not call
`openbase-coder computer-use start` unless the user explicitly asks the AI to
control the screen.

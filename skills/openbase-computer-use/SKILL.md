---
name: openbase-computer-use
description: >-
  Use this skill when an agent needs to operate the user's visible desktop
  through Computer Use tools. Routes to the correct path based on agent
  environment and operating system.
version: 0.5.0
---

# Openbase Computer Use

Computer use is intentionally visible. The user can stop the operation at any
time. Treat any user request to stop, pause, abort, or interrupt as an immediate
instruction to stop issuing further desktop actions.

## What is supported where

| Capability | Codex (macOS) | Codex (Linux DevSpace) | Claude Code via Openbase (macOS) | Claude Code via Openbase (Linux DevSpace) |
|---|---|---|---|---|
| Computer control | ✓ `$computer-use` plugin tools | ✓ `openbase-coder computer-use` run | ✓ `mcp__openbase-computer-use__*` | ✓ `mcp__openbase-computer-use__*` |
| Chrome control | ✗ not supported | ✗ not supported | ✓ `mcp__claude-in-chrome__*` (Chrome plugin toggle) | ✗ not supported |

Computer control works everywhere; Chrome control exists only for Claude Code
sessions on macOS. Never attempt Chrome control from a Codex agent or on a
Linux DevSpace — if a user asks for it there, explain it is only available on
the Claude Code backend on a Mac.

## Which path to follow

Determine your environment and follow the corresponding sub-file:

| Environment | File |
|---|---|
| Codex agent running on macOS | [codex-macos.md](codex-macos.md) |
| Interactive Claude Code on macOS | [claude-code-macos.md](claude-code-macos.md) |
| Openbase Coder Claude session (macOS or Linux) | [openbase-claude.md](openbase-claude.md) |
| Codex or other agent on Linux | [linux.md](linux.md) |

**How to tell which environment you are in:**

- If you have native `mcp__computer-use__*` tools available (e.g.
  `request_access`, `screenshot`, `left_click`) and you are Claude Code → use
  [claude-code-macos.md](claude-code-macos.md).
- If you have `mcp__openbase-computer-use__*` tools available (Claude Code
  sessions started by Openbase Coder on either OS — the built-in suite never
  attaches to headless sessions) → use [openbase-claude.md](openbase-claude.md).
- If neither MCP tool set is present in a Claude Code session, the Computer
  Use plugin is off — point the user at console → Settings → Coding backend
  (Claude Code) → Computer Use instead of improvising desktop control.
- If you are a Codex agent: [codex-macos.md](codex-macos.md) on macOS,
  [linux.md](linux.md) on Linux.

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

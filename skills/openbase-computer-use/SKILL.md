---
name: openbase-computer-use
description: >-
  Use this skill when an agent needs to operate the user's visible desktop
  through Computer Use tools. Routes to the correct path based on agent
  environment and operating system.
version: 0.2.0
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

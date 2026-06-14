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

## Manual iOS Remote Control Screen Share

Use this when the user wants to control the Linux desktop manually from the
Openbase iOS app. This is not an AI computer-use run. Do not call
`openbase-coder computer-use start` unless the user explicitly asks the AI to
control the screen.

Requirements:

- This flow is for Linux Openbase DevSpace desktops.
- The Openbase iOS app must be in an active LiveKit call connected to the same
  backend/room.
- The Linux companion must be available from the base image. If the companion is
  not already running, the start path below launches it.

Start the screen share for manual iOS control:

```bash
python - <<'PY'
from openbase_coder_cli.cli.computer_use import CompanionClient, _load_companion_session

client = CompanionClient()
client.ensure_running()
client.start_screen_share(_load_companion_session(""))
print("Openbase screen share started for manual iOS control.")
PY
```

The iOS app also calls the local backend's `/api/livekit-companion-start/`
endpoint when room control connects. Do not use raw `curl` for that endpoint
unless you already have the right local auth headers; use the Python helper
above for agent-initiated starts.

Stop the manual screen share:

```bash
python - <<'PY'
from openbase_coder_cli.cli.computer_use import CompanionClient

CompanionClient().stop_screen_share()
print("Openbase screen share stopped.")
PY
```

While the share is active, the user can open the full-screen screen-share view
in the iOS app, enable Remote, and control the desktop with the trackpad,
scroll strip, click buttons, keyboard field, and shortcut keys. The companion
only accepts remote-control input after the iOS user enables Remote, and it
disables manual remote control when the screen share stops.

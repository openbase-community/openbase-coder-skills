# Openbase Computer Use — Linux

On Linux Openbase DevSpace desktops, use the `openbase-coder computer-use` CLI
commands to perform desktop control. The native `mcp__computer-use__*` MCP
tools are macOS-only and are not available in Linux environments.

## CLI commands

```bash
openbase-coder computer-use start    # begin a desktop-control session
openbase-coder computer-use steer    # send an action or instruction
openbase-coder computer-use queue    # queue an action
openbase-coder computer-use status   # check session status
openbase-coder computer-use interrupt  # stop the current action
```

Refer to the CLI implementation at `cli/openbase_coder_cli/cli/computer_use.py`
for the full command reference and options.

## Key combo conventions (Linux)

- Use `ctrl` (not `cmd`) for most shortcuts: `ctrl+c`, `ctrl+n`.
- App launcher / run dialog: `super+space` or `alt+F2` depending on the desktop
  environment.
- Key chords use `super` for the Meta/Windows key.

## Notes on X11 vs Wayland

- The supported target is the Openbase DevSpace AMI: Ubuntu desktop, Xorg, and
  NICE DCV.
- The CLI expects X11 tooling (`xdotool`, `scrot` or `gnome-screenshot`, and
  ImageMagick). Arbitrary Wayland desktops are not supported by this path.

## Safety Rules

- The user can stop at any time. On any stop/abort/pause signal, call
  `openbase-coder computer-use interrupt` immediately.
- Keep actions narrow. One interaction, then verify with a status or snapshot
  check.
- Do not type secrets, credentials, API keys, or private data.

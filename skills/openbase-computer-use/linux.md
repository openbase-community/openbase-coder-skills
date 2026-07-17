# Openbase Computer Use — Linux (Codex and other non-Claude agents)

On Linux Openbase DevSpace desktops, use the `openbase-coder computer-use` CLI
commands to perform desktop control. The native `mcp__computer-use__*` MCP
tools are macOS-only and are not available in Linux environments.

Claude Code sessions started by Openbase Coder are the exception: when
`mcp__openbase-computer-use__*` tools are available, follow
[openbase-claude.md](openbase-claude.md) instead of this CLI path — the same
Linux companion serves those tools directly. Chrome control is not available
on Linux for any agent.

## CLI commands

```bash
openbase-coder computer-use start    # begin a desktop-control session
openbase-coder computer-use steer    # send an action or instruction
openbase-coder computer-use queue    # queue an action
openbase-coder computer-use status   # check session status
openbase-coder computer-use interrupt  # stop the current action
```

For manual iOS remote control (no AI run), share the screen with:

```bash
openbase-coder computer-use screen-share start  # share this desktop to iOS
openbase-coder computer-use screen-share stop   # stop sharing
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
- The companion auto-detects the user's X display (on DevSpaces the DCV
  session may be `:1` while GDM's greeter holds `:0`). If detection picks the
  wrong display, set `OPENBASE_COMPUTER_USE_DISPLAY` explicitly and check
  `openbase-coder computer-use status`.

## Safety Rules

- The user can stop at any time. On any stop/abort/pause signal, call
  `openbase-coder computer-use interrupt` immediately.
- Keep actions narrow. One interaction, then verify with a status or snapshot
  check.
- Do not type secrets, credentials, API keys, or private data.

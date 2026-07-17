# Openbase Computer Use — Claude Code in Openbase Coder sessions (macOS and Linux)

Headless Claude Code sessions started by Openbase Coder (the dispatcher and
Super Agents threads) never get Claude Code's built-in `mcp__computer-use__*`
suite — it is interactive-only. They use the Openbase-provided
`mcp__openbase-computer-use__*` tools instead, which work the same on a Mac
(served by the Openbase Coder desktop app) and on a Cloud DevSpace Linux
desktop (served by the Linux companion).

Use this path when you have `mcp__openbase-computer-use__*` tools available.
If you have the native `mcp__computer-use__*` tools instead, you are in an
interactive Claude Code session — use
[claude-code-macos.md](claude-code-macos.md).

## When the tools are available

- The Computer Use plugin must be enabled: console → Settings → Coding
  backend (Claude Code) → Computer Use, then recreate the dispatcher. If the
  `mcp__openbase-computer-use__*` tools are missing entirely, this toggle is
  the fix — tell the user rather than improvising desktop control another
  way.
- On macOS, the Openbase Coder desktop app must be running. If tool calls
  report that the desktop app is not running, ask the user to start it.
- On Linux (Cloud DevSpace), the companion launches when screen sharing
  starts; if tool calls report the companion is not running, start the
  screen share (below) and retry.

## Screen-Share Rule

Screen sharing is enforced, not optional: every desktop-control action is
refused unless the user's screen is being shared, so the user always sees
what you do. Before the first action, start the share with the command for
the host OS:

macOS:

```
openbase-coder desktop screen-share start
```

Linux DevSpace:

```
openbase-coder computer-use screen-share start
```

Stop the share (`... screen-share stop`) when the requested desktop work is
finished. If an action fails with a screen-share-inactive error, start the
share and retry rather than retrying blindly.

## Foreground Rule

ALWAYS foreground the app you are about to interact with. Call
`mcp__openbase-computer-use__open_application` before the first action
targeting an app, and again any time focus shifts (dialogs, app switches,
actions that open another app).

## Tool reference

| Task | Tool |
|---|---|
| Take a screenshot | `mcp__openbase-computer-use__screenshot` |
| Bring app to front / open app | `mcp__openbase-computer-use__open_application` |
| Left click | `mcp__openbase-computer-use__left_click` |
| Double click | `mcp__openbase-computer-use__double_click` |
| Right click | `mcp__openbase-computer-use__right_click` |
| Move mouse | `mcp__openbase-computer-use__mouse_move` |
| Type text | `mcp__openbase-computer-use__type` |
| Press a key or chord | `mcp__openbase-computer-use__key` |
| Scroll | `mcp__openbase-computer-use__scroll` |
| Get cursor position | `mcp__openbase-computer-use__cursor_position` |
| Batch sequential actions | `mcp__openbase-computer-use__computer_batch` |

Click coordinates map 1:1 to the pixels of the most recent screenshot on both
OSes — always screenshot before acting, and re-screenshot after actions that
change the screen. Use `computer_batch` when the full input sequence is
predictable.

## Key combo conventions

- macOS: use `cmd` (not `ctrl`): `cmd+c`, `cmd+n`, `cmd+space`.
- Linux: use `ctrl`: `ctrl+c`, `ctrl+n`. (`cmd` combos are tolerated — they
  are remapped to `ctrl` — but prefer `ctrl` for clarity.)
- Combos are single strings: `key(combo="cmd+shift+t")`.

## Browser work (macOS only)

If the Chrome plugin is enabled for the Claude Code backend (console →
Settings → Coding backend → Chrome), macOS sessions also get the native
`mcp__claude-in-chrome__*` browser tools. Prefer those for anything inside a
web page — clicking, forms, reading page content — and reserve the
`mcp__openbase-computer-use__*` tools for native apps and OS-level work.

Chrome control is not available on Linux DevSpaces; use
`mcp__openbase-computer-use__*` against a browser window there only when the
task truly requires it.

## Workflow

1. Identify the target app and the exact visible outcome.
2. Start the screen share (or confirm it is active).
3. Foreground the target app with `open_application`.
4. Take a `screenshot` to confirm state before acting.
5. Interact with the smallest visible action; re-screenshot when the result
   matters.
6. Re-foreground after any focus-shifting action.
7. Verify the outcome from a final screenshot before reporting done.
8. Stop the screen share once the desktop work is complete.

## Safety Rules

- The user can stop at any time — stopping the screen share cuts off all
  control immediately. Treat any stop/pause/abort request as an instruction to
  stop issuing actions.
- Keep actions narrow and explicit. One interaction, then verify.
- Do not type secrets, credentials, API keys, or private data.
- Do not start a second desktop-control flow while one is active.
- macOS permissions: the desktop companion needs Screen Recording and
  Accessibility grants. If an action fails with an Accessibility error, relay
  the System Settings instructions to the user instead of retrying.
- Linux readiness: the DevSpace desktop needs X11 tooling (xdotool,
  ImageMagick, scrot). If an action reports missing tools, relay that to the
  user — it is an environment problem, not a retry problem.

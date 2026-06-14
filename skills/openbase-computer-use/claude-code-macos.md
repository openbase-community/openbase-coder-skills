# Openbase Computer Use — Claude Code on macOS

Claude Code has a native `mcp__computer-use__*` tool suite. Use those tools
directly — do not use the `openbase-coder computer-use` CLI commands.

There is no separate screen-share session to manage. Access is granted via
`request_access` at the start of each session.

## Setup: request access first

Before any desktop action, call `mcp__computer-use__request_access` with the
list of applications you need to control and a one-sentence reason.

```
request_access(apps=["Typora", "Slack"], reason="Open Typora and paste a draft")
```

The user sees a system dialog and approves or denies per app. Granted apps
persist for the session. If mid-task you need an additional app, call
`request_access` again — previously granted apps remain granted.

Re-check `list_granted_applications` if you are unsure what is currently
allowed.

## Foreground Rule

ALWAYS foreground the app you are about to interact with before interacting.
Call `mcp__computer-use__open_application` before the first action targeting
an app, and again any time focus shifts away (dialogs, app switches, etc.).

Failure to do this causes typing or clicks to land in the wrong app.

Preferred pattern:

1. Call `open_application` for the target app.
2. Call `screenshot` to confirm the app is frontmost.
3. Perform the smallest visible interaction needed.
4. After any action that might shift focus (dialog dismiss, keyboard shortcut
   that opens another app, etc.), call `open_application` again before the
   next interaction.

## Tool reference

| Task | Tool |
|---|---|
| Request desktop access | `mcp__computer-use__request_access` |
| Bring app to front | `mcp__computer-use__open_application` |
| Take a screenshot | `mcp__computer-use__screenshot` |
| Zoom into a region | `mcp__computer-use__zoom` |
| Left click | `mcp__computer-use__left_click` |
| Double click | `mcp__computer-use__double_click` |
| Right click | `mcp__computer-use__right_click` |
| Type text | `mcp__computer-use__type` |
| Press a key or chord | `mcp__computer-use__key` |
| Scroll | `mcp__computer-use__scroll` |
| Batch sequential actions | `mcp__computer-use__computer_batch` |
| Move mouse | `mcp__computer-use__mouse_move` |
| Get cursor position | `mcp__computer-use__cursor_position` |

Use `computer_batch` whenever you can predict the full sequence in advance —
it eliminates round trips and is significantly faster than individual calls.

## Key combo conventions (macOS)

- Use `cmd` (not `ctrl`) for macOS shortcuts: `cmd+c`, `cmd+n`, `cmd+space`.
- Spotlight: `cmd+space`.
- New document: `cmd+n`.
- Quit app: `cmd+q` (requires `systemKeyCombos` grant).

Request `systemKeyCombos=True` in `request_access` only when you need
quit-app, switch-app, or lock-screen level combos.

## App tier restrictions

Some apps are granted at a restricted tier:

- **Browsers** (Chrome, Safari, etc.) → tier **"read"**: visible in
  screenshots, clicks and typing blocked. Use the `mcp__claude-in-chrome__*`
  tools for browser interaction instead.
- **Terminals and IDEs** (Terminal, VS Code, Cursor, etc.) → tier **"click"**:
  visible and left-clickable, but typing and right-click are blocked. Use the
  Bash tool for shell commands instead.
- **Everything else** → tier **"full"**: no restrictions.

If a tool call errors because the frontmost app is at a restricted tier, switch
to the appropriate alternative tool rather than retrying.

## Workflow

1. Identify the target app and exact visible outcome.
2. Call `request_access` for all apps needed upfront.
3. Call `open_application` to foreground the target app.
4. Call `screenshot` to confirm state before acting.
5. Interact using the tools above. Batch predictable sequences with
   `computer_batch`.
6. After any focus-shifting action, call `open_application` again before the
   next interaction.
7. Call `screenshot` to verify the requested outcome is visible.
8. Report done with what was accomplished and any notable state (e.g. unsaved
   document, dialog left open).

## Safety Rules

- The user can stop at any time. Stop immediately on any stop/abort/pause
  signal.
- Keep actions narrow and explicit. One interaction, then verify.
- Do not type secrets, credentials, API keys, or private data.
- Do not start a second desktop-control flow while one is active.
- Coordinates always refer to the most recent full-screen screenshot. After
  zooming with `zoom`, use full-screen coordinates for subsequent clicks.

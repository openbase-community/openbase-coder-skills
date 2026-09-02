# Openbase Computer Use — Codex on macOS

Use the Computer Use tools available in the current agent environment to control
the user's visible Mac desktop. Do not use the `openbase-coder computer-use`
command to perform desktop control when tool-based Computer Use is available.

## Screen-Share Rule

Require Openbase screen sharing only when the user is communicating through an active voice interface, indicated by `<voice>` tags. In an ordinary text session, use the attached Computer Use tools directly; do not require or start a LiveKit screen share.

During an active `<voice>` session, screen sharing wraps the desktop-control workflow: confirm it before the first Computer Use action, keep it active while work is in progress, and stop or release it after the requested work finishes. Tool-based Computer Use does not automatically manage the Openbase Coder screen share, so the agent owns those checks in voice sessions.

If screen sharing is unavailable during an active `<voice>` session, do not click, type, scroll, drag, press keys, or set UI values. Ask the user to start screen sharing or use the available Openbase Coder screen-share mechanism first. This restriction does not apply to ordinary text sessions.

## Foreground Rule

ALWAYS foreground the app you are about to interact with before you interact
with it. This applies before clicking, typing, scrolling, dragging, pressing
keys, reading UI state for an action, or setting any control value.

Foregrounding is per application, not per session. If the application being
interacted with changes, foreground the new target app before interacting with
it. If an action opens or switches to another app, dialog host, browser window,
media player, system prompt, or helper app, call foregrounding again for that
new target before the next click, keypress, scroll, drag, read-for-action, or
value change.

Preferred pattern:

1. If this is an active `<voice>` session, confirm screen sharing is active.
2. Use the available Computer Use app-state/snapshot tool for the target app.
3. If the target app is not visibly foregrounded, bring it to the foreground
   with the available app activation or OS focus mechanism.
4. Verify the app is foregrounded.
5. Perform the smallest visible interaction needed.
6. Re-check app state after interactions that may change focus, navigation,
   playback, or the active application. If the target app changed, foreground
   the new app before continuing.

If a Computer Use interaction fails because the tool session is inactive, use
the current environment's visible desktop-control fallback only after
foregrounding the target app again. Keep these fallbacks narrow and visible.

## When To Use

Use this path when:

- You are a Codex agent and `openbase-coder computer-use` CLI commands are
  available in your environment.
- The user asks to use Openbase Coder computer use, screen control, or computer
  control to operate their Mac.
- The user asks to test keyboard, mouse, or UI input through visible Computer
  Use tools.
- The user asks to inspect or manipulate a visible desktop app.

Do not use this path for ordinary shell automation, browser automation, or
repo edits that do not require operating the user's visible desktop.

Chrome control is not supported from Codex agents on any OS. If the user asks
for browser control, explain it is available only on the Claude Code backend
on macOS (see the support matrix in [SKILL.md](SKILL.md)).

## Safety Rules

- During an active `<voice>` session, computer control must happen only while screen sharing is active. In a text session, use the available Computer Use tools without starting a LiveKit screen share.
- The user can stop at any time. If they say stop, abort, pause, interrupt, or
  indicate the control is doing the wrong thing, immediately stop taking desktop
  actions and report what was stopped.
- Keep actions narrow and explicit. Prefer one visible interaction at a time,
  followed by a state check when the result matters.
- Do not ask computer use to type secrets, credentials, API keys, or private
  data.
- Do not start another desktop-control flow while one is already controlling.
- Watch for focus problems: typing goes into whatever app is focused. Never ask
  the user to focus the target app when you can foreground it yourself with the
  available tools.

## Workflow

1. Identify the target app and the exact visible outcome the user requested.
2. If this is an active `<voice>` session, confirm screen sharing is active before doing anything else.
3. Foreground the target app.
4. Read the app state with the available Computer Use tool.
5. Interact using the available Computer Use click, type, press-key, scroll,
   drag, select, or set-value tools.
6. Foreground the target app again before each later interaction, especially
   after clicks, app switches, dialogs, or playback/navigation changes. If the
   interaction target changes from one app to another, foreground the new app
   first.
7. Verify the requested outcome from visible app state before reporting done.
8. If screen sharing was started for an active `<voice>` session, end or release it after the requested Computer Use work finishes.
9. If behavior is unclear, inspect recent logs only when they are directly
   relevant. Limit log output because Openbase logs can be large.

## Notes

- Prefer the Computer Use MCP/plugin tools exposed to the agent session over
  shell commands.
- Do not invoke `openbase-coder computer-use start`, `steer`, `queue`,
  `status`, or `interrupt` to perform tool-based desktop control.
- Use shell or OS-level app activation only as a foregrounding fallback when the
  Computer Use tool cannot bring the app forward itself.
- The implementation is in the Openbase Coder workspace:
  - CLI: `cli/openbase_coder_cli/cli/computer_use.py`
  - Swift companion: `desktop/companion/livekit-swift-example/Multiplatform`
  - iOS screen-share UI: `ios/Openbase/Vendor/OpenbaseAgentUI/Calling`

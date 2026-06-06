---
name: openbase-computer-use-cli
description: >-
  Use this skill when an agent needs to start, steer, interrupt, test, or
  reason about Openbase Coder computer-use CLI sessions that control the user's
  Mac through the LiveKit screen-share companion.
version: 0.1.0
---

# Openbase Computer Use CLI

Use the Openbase Coder `computer-use` CLI to control the user's Mac through the
LiveKit screen-share companion.

Computer use is intentionally tied to screen sharing. A LiveKit screen share is
required before control can start, and the user can stop the operation at any
time. Treat any user request to stop, pause, abort, or interrupt as immediate
instruction to run the interrupt command.

There are two control modes:

- `computer-use`: OpenAI Computer Use controls the visible macOS desktop while
  the full display is shared.
- `claude-chrome`: Claude Code runs with `--chrome` while only one managed
  Chrome window is shared.

## When To Use

Use this skill when:

- The user asks to use Openbase Coder computer use, screen control, computer
  control, or the LiveKit companion to operate their Mac.
- The user asks to use Claude with Chrome while sharing a single Chrome window.
- The user asks to test keyboard/mouse input through the computer-use CLI.
- The user asks to steer, queue, interrupt, stop, or inspect an active
  computer-use run.

Do not use this skill for ordinary shell automation, browser automation, or
repo edits that do not require operating the user's visible desktop.

## Safety Rules

- Computer control must only happen while LiveKit screen sharing is active.
- The CLI start path should start screen sharing before starting control; if
  screen sharing cannot start, do not attempt control another way.
- The user can stop at any time. If they say stop, abort, pause, interrupt, or
  indicate the control is doing the wrong thing, immediately run:

  ```bash
  openbase-coder computer-use interrupt
  ```

  For Claude Chrome runs, use:

  ```bash
  openbase-coder claude-chrome abort
  ```

- Keep instructions narrow and explicit. Use at least `--max-steps 20` for
  computer-use runs, even for simple tests, unless the user explicitly requests
  a lower cap.
- Do not ask computer use to type secrets, credentials, API keys, or private
  data.
- Do not start another computer-use run while one is already controlling.
- Watch for focus problems: the tool can type into whatever app is focused.
  When testing keystrokes, ask the user to focus the target app first.

## Workflow

1. Check status before starting or steering:

   ```bash
   openbase-coder computer-use status
   ```

2. Start a run with a short, concrete instruction:

   ```bash
   openbase-coder computer-use start --max-steps 8 "Open System Settings."
   ```

   The CLI discovers the active LiveKit room, launches the companion if needed,
   starts screen sharing, then starts computer use. Screen sharing is stopped
   automatically after completion.

3. If the room resolver is confused and the correct room name is known, pass it
   explicitly:

   ```bash
   openbase-coder computer-use start --room ROOM_NAME --max-steps 8 "..."
   ```

4. Steer an active run instead of starting a second one:

   ```bash
   openbase-coder computer-use steer "Use Spotlight only. Do not type into the current chat."
   ```

5. Queue a follow-up instruction when the user wants chained work after the
   current task finishes:

   ```bash
   openbase-coder computer-use queue "Then open the saved file and verify it."
   ```

   Queueing requires an active computer-use run. The queued instruction starts
   before the CLI stops screen sharing, so do not start a separate run for the
   next step.

6. Interrupt immediately when the user asks or the run is acting in the wrong
   app:

   ```bash
   openbase-coder computer-use interrupt
   ```

7. Inspect recent Openbase Coder and LiveKit companion logs if behavior is
   unclear. Limit log output because these logs can be large.

## Claude Chrome Mode

Use this when the user wants a browser-limited control surface instead of full
desktop control. It opens a single Chrome window, shares that window to LiveKit,
and starts Claude Code with `--chrome`.

Start:

```bash
openbase-coder claude-chrome start --url about:blank --max-turns 8 "Open the target site and inspect the page."
```

Steer:

```bash
openbase-coder claude-chrome steer "Try the search field instead."
```

Queue:

```bash
openbase-coder claude-chrome queue "Then summarize the page contents."
```

Abort:

```bash
openbase-coder claude-chrome abort
```

Steering in this mode replaces the current Claude browser process with a
continued `claude -p --chrome --continue` prompt in the same shared Chrome
window. Queueing does not interrupt the current Claude process; it appends a
continued `claude -p --chrome --continue` prompt that starts after the current
one exits, while the same Chrome window remains shared.

## Notes

- Assume `openbase-coder` is installed on `PATH`; do not invoke it through a
  workspace checkout or package-manager wrapper in this skill.
- The companion should be launched through the CLI's configured discovery path;
  do not hard-code a global companion app location in this skill.
- The companion reads `OPENAI_API_KEY` from the Openbase Coder environment; do
  not use or create alternate ad hoc key files.
- The implementation is in the Openbase Coder workspace:
  - CLI: `cli/openbase_coder_cli/cli/computer_use.py`
  - Claude Chrome CLI: `cli/openbase_coder_cli/cli/claude_chrome.py`
  - Swift companion: `desktop/companion/livekit-swift-example/Multiplatform`
  - iOS screen-share UI: `ios/Openbase/Vendor/OpenbaseAgentUI/Calling`

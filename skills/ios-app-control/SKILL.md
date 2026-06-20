---
name: ios-app-control
description: >-
  Use this skill when the user asks an agent to control the foreground Openbase
  iOS app, including opening a URL or deep link, muting the current call, or
  unmuting the current call, switching to the debug LiveKit voice test call, or
  switching back to the regular developer call.
version: 0.1.0
---

# iOS App Control

Use the Openbase Coder CLI to send commands to the foreground Openbase iOS app.
The app must be open, authenticated, and connected to the local Openbase Coder
server. Commands are best-effort; if the app is closed or disconnected, nothing
on the phone will happen.

## Commands

Open a web link or deep link on iOS:

```bash
openbase-coder user ios open-url "https://example.com"
openbase-coder user ios open-url "openbase://example"
```

Mute the active iOS voice call:

```bash
openbase-coder user ios mute
```

Unmute the active iOS voice call:

```bash
openbase-coder user ios unmute
```

End the current normal iOS voice call, switch to the debug LiveKit voice test
tab, and start the voice test call from the already-filled debug fields:

```bash
openbase-coder user ios start-livekit-voice-test
```

End the debug LiveKit voice test call, switch back to the normal Call tab, and
start the regular developer call:

```bash
openbase-coder user ios start-developer-call
```

## Manual Desktop Screen Control

Use this when the user wants to manually control a desktop from the Openbase iOS
app through an active Openbase screen share. Manual iOS control is
cross-platform: it can control any Openbase screen-share companion that
implements the `openbase.remote_control.*` LiveKit messages. This is separate
from AI computer-use. Do not start an AI computer-use run unless the user
explicitly asks the AI to operate the screen.

How the iOS side works:

- The iOS app must be open, authenticated, and in an active LiveKit call.
- On Linux DevSpaces, when room control connects, the iOS app asks the local
  backend to start the Linux screen-share companion through
  `/api/livekit-companion-start/`.
- On macOS, the screen share is provided by the Electron app's bundled LiveKit
  companion instead of the Linux `openbase-coder computer-use` CLI companion.
- Once the remote screen-share tile appears, the user opens it full screen and
  taps Remote to enable manual control.
- The full-screen screen-share view supports pinch zoom, drag-to-pan, a
  trackpad surface, a scroll strip, click buttons, typed text, and shortcut
  keys.

If the user asks the agent to start screen sharing for manual iOS control from
the Linux desktop side, use the Openbase computer-use companion without starting
the AI runner. This CLI path is Linux-only:

```bash
python - <<'PY'
from openbase_coder_cli.cli.computer_use import CompanionClient, _load_companion_session

client = CompanionClient()
client.ensure_running()
client.start_screen_share(_load_companion_session(""))
print("Openbase screen share started for manual iOS control.")
PY
```

Stop that Linux DevSpace manual screen share:

```bash
python - <<'PY'
from openbase_coder_cli.cli.computer_use import CompanionClient

CompanionClient().stop_screen_share()
print("Openbase screen share stopped.")
PY
```

## Notes

- Only use this for explicit user requests to control their phone.
- `open-url` requires a URL scheme. It supports normal web URLs and custom deep
  links, but rejects `data:`, `file:`, and `javascript:` URLs.
- Mute and unmute require an active Openbase voice call in the iOS app.
- `start-livekit-voice-test` assumes the LiveKit voice test fields are already
  filled in on the phone.
- `start-developer-call` starts the normal Openbase dispatcher/developer call.
- Manual desktop screen control requires an active Openbase screen share. If the
  user cannot see the remote screen in iOS, start the platform-appropriate
  companion screen share first; if the user cannot control it, make sure they
  tapped Remote in the full-screen screen-share view.

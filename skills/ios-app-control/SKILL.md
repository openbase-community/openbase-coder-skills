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

Ask the foreground iOS app to upload its recent retained diagnostics logs to the
local Openbase Coder server:

```bash
openbase-coder user ios upload-logs
openbase-coder user ios upload-logs --limit 500
```

Uploaded entries are appended by the local server to
`~/.openbase/logs/ios-app.log`.

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
app. Start the desktop screen share:

```
openbase-coder desktop screen-share start
```

Stop the desktop screen share:

```
openbase-coder desktop screen-share stop
```

After the screen-share tile appears in iOS, the user opens it full screen and
taps Remote to enable manual control.

This is separate from AI computer-use. Do not start an AI computer-use run
unless the user explicitly asks the AI to operate the screen.

## Notes

- Only use this for explicit user requests to control their phone.
- `open-url` requires a URL scheme. It supports normal web URLs and custom deep
  links, but rejects `data:`, `file:`, and `javascript:` URLs.
- Mute and unmute require an active Openbase voice call in the iOS app.
- `upload-logs` requires the iOS app to be foregrounded or connected to the
  app-control WebSocket. It reuses the app's diagnostics uploader and does not
  require the user to tap the upload button.
- `start-livekit-voice-test` assumes the LiveKit voice test fields are already
  filled in on the phone.
- `start-developer-call` starts the normal Openbase dispatcher/developer call.
- Manual desktop screen control requires an active Openbase screen share. If the
  user cannot see the remote screen in iOS, run the screen-share start command;
  if the user cannot control it, make sure they tapped Remote in the full-screen
  screen-share view.

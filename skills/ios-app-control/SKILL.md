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

## Notes

- Only use this for explicit user requests to control their phone.
- `open-url` requires a URL scheme. It supports normal web URLs and custom deep
  links, but rejects `data:`, `file:`, and `javascript:` URLs.
- Mute and unmute require an active Openbase voice call in the iOS app.
- `start-livekit-voice-test` assumes the LiveKit voice test fields are already
  filled in on the phone.
- `start-developer-call` starts the normal Openbase dispatcher/developer call.

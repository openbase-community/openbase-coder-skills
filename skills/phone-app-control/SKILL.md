---
name: phone-app-control
description: >-
  Use this skill when the user asks an agent to do something in the Openbase
  app on their phone (iOS or Android), including opening a URL or deep link,
  muting or unmuting the current call, and, on iOS only, switching to the
  debug LiveKit voice test call, switching back to the regular developer call,
  or driving call state for an authorized voice test.
version: 0.3.0
---

# Phone App Control

Use the Openbase Coder CLI to send commands to the Openbase app on the user's
phone. The same commands work on iOS and Android. The app must be signed in and
connected to this computer. Commands are best-effort; if the app is closed or
disconnected, nothing on the phone will happen.

When you tell the user what you did, say "on your phone" (or name the phone's
actual platform if you know it). Do not call it the iOS app unless you are
using one of the iOS-only commands below.

## Commands for any phone

Open a web link or deep link on the phone:

```bash
openbase-coder user phone open-url "https://example.com"
openbase-coder user phone open-url "openbase://example"
```

If the app is in front, the link opens right away. If it is in the background
or closed, the phone gets a notification, and the link opens when the user taps
it. Tell the user to tap the notification in that case.

Mute or unmute the active voice call:

```bash
openbase-coder user phone mute
openbase-coder user phone unmute
```

## iOS-only commands

These commands have no Android support yet. Use them only for an iPhone.

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

End the debug LiveKit voice test call, switch back to the new-chat home screen, and start the regular developer call:

```bash
openbase-coder user ios start-developer-call
```

## Call controls with resulting state

Use these commands for an explicitly authorized iOS voice test. They reuse the same authenticated local HTTP API and foreground app-control WebSocket as the commands above; the existing app-control channel is available in both normal and field-test builds. Field tests must still use the dedicated field-test variant.

```bash
openbase-coder user ios start-call dispatcher
openbase-coder user ios start-call <thread-id>
openbase-coder user ios set-speaker on
openbase-coder user ios set-speaker off
openbase-coder user ios end-call
```

The wire actions are `start_call` (`thread_id`: an existing thread id or `dispatcher`), `set_speaker` (`speaker`: boolean), and `end_call`. Each CLI command prints JSON containing `delivered`, `applied`, and `call_state` with boolean `connected`, `muted`, `speaker`, and `active` fields. `active` also covers connection preparation and CallKit teardown. These commands acknowledge after handling, unlike legacy receipt-only commands. `speaker` reports the actual built-in speaker output route, and `muted` reports the LiveKit microphone state. A route request that fails to take effect returns `applied: false`; never infer speaker-on from command delivery alone.

`start-call` requires microphone permission to have already been granted and refuses to replace an existing call. A thread target connects the normal call then uses the existing voice-route transfer API; `dispatcher` starts the normal dispatcher call. Microphone permission failures return immediately without opening a permission sheet. Token, room, or CallKit failures return their error and tear down the failed start; connection timeout or failed thread selection also disconnects locally. `end-call` is idempotent, cancels pending starts, and ends both normal and debug LiveKit test calls even if CallKit never delivered the start action. Require `applied: true`, `connected: false`, and `active: false` before considering hang-up complete. A failed or timed-out command exits nonzero; missing state means an old app or no completion acknowledgement, never success. Run `end-call` to clean up an unconfirmed start before retrying. Do not automatically retry `start-call`.

The server waits up to 45 seconds for completion and the CLI allows 60 seconds. Both the controlling runtime and iOS app must contain these commands; updating only the phone is insufficient. A retained older runtime must be updated through its authorized installation workflow before it can accept the new actions.

For acoustic testing, start the call, explicitly set speaker on, require `applied: true` and `call_state.speaker: true`, and preserve any screenshot verification required by the field-testing procedure before speaking. Hang up in the same test step and verify the returned disconnected/inactive state before waiting, building, or ending the session. State acknowledgements do not waive a screenshot requirement. No Android parity is provided by these commands.

## Manual Desktop Screen Control

Use this when the user wants to manually control a desktop from the Openbase
phone app. Start the desktop screen share:

```
openbase-coder desktop screen-share start
```

Stop the desktop screen share:

```
openbase-coder desktop screen-share stop
```

After the screen-share tile appears in the app, the user opens it full screen and
taps Remote to enable manual control.

This is separate from AI computer-use. Do not start an AI computer-use run
unless the user explicitly asks the AI to operate the screen.

## Notes

- Only use this for explicit user requests to control their phone.
- `open-url` requires a URL scheme. It supports normal web URLs and custom deep
  links, but rejects `data:`, `file:`, and `javascript:` URLs.
- Mute and unmute require an active Openbase voice call in the phone app.
- A command is applied only when the phone acknowledges it. The CLI prints `command delivered` on success; `published (unconfirmed)` plus a non-zero exit means no phone app received it (not connected, backgrounded, or signed out). Report that to the user as **not done**; never say the phone was muted or unmuted unless the command was delivered. `open-url` instead says whether it opened the link or sent a notification to tap.
- `upload-logs` requires the iOS app to be foregrounded or connected to the
  app-control WebSocket. It reuses the app's diagnostics uploader and does not
  require the user to tap the upload button.
- `start-livekit-voice-test` assumes the LiveKit voice test fields are already
  filled in on the phone.
- `start-developer-call` starts the normal Openbase dispatcher/developer call.
- Manual desktop screen control requires an active Openbase screen share. If the
  user cannot see the remote screen in the app, run the screen-share start command;
  if the user cannot control it, make sure they tapped Remote in the full-screen
  screen-share view.

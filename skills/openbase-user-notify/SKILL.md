---
name: openbase-user-notify
description: >-
  Use this skill when an agent needs to reach the Openbase user out of band —
  speak into an active call, send a deep-linked push notification to their
  iPhone, or ring their phone with an inbound call — especially when the user is
  not currently on a voice call. Covers openbase-coder user say / notify / call
  and the deep-link destination vocabulary.
version: 0.1.0
---

# Notify the Openbase User

An agent can reach the user three ways, in increasing urgency. Prefer the least
intrusive one that fits. All commands run through the local `openbase-coder`
CLI and send to whichever iPhone the user has signed in and registered.

## 1. Speak in the active call — `user say`

Speaks an announcer message into the live voice session:

```bash
openbase-coder user say AGENT_NAME "Your message here"
```

If there is **no active voice session**, `say` automatically falls back to a
push notification: it prints a note to stderr that it is falling back and then
sends the message as an iPhone push that deep-links to the agent's thread (or the
dispatcher). So `user say` is always safe to use — the user gets the message
whether or not they are on a call.

## 2. Deep-linked push notification — `user notify`

Send a notification that opens a specific screen when tapped. Works whether or
not the user is on a call, and delivers even when the app is backgrounded or
closed (via APNs):

```bash
# Simple heads-up (opens the dispatcher by default)
openbase-coder user notify --body "Finished the refactor."

# Deep-link to a specific coding thread
openbase-coder user notify --body "Tests are green." \
  --destination threads --thread-id THREAD_ID

# Deep-link to a specific report
openbase-coder user notify --body "Security review ready." \
  --destination reports --report-id REPORT_ID

# Deep-link to approvals with a custom title
openbase-coder user notify --title "Approval needed" \
  --body "Deploy to production?" --destination approvals
```

Options:

- `--body` (required): notification body text.
- `--title`: notification title (default `Openbase Coder`).
- `--destination`: which screen to open on tap (see vocabulary below).
- `--thread-id` / `--report-id`: deep-link target for `threads` / `reports`.
- `--param KEY=VALUE`: extra deep-link params, repeatable.

## 3. Ring the phone — `user call`

Place an inbound call that rings the user's iPhone via CallKit (full-screen
incoming call, even from background/closed). Answering joins the LiveKit room:

```bash
# Ring into a new room
openbase-coder user call --caller-name "Dispatcher"

# Ring into an existing room with a specific dispatch agent
openbase-coder user call --room ROOM_NAME --agent-name livekit-agent \
  --caller-name "Build Agent"
```

Only use `call` when a live voice conversation is genuinely warranted (the agent
was explicitly told to call, or something needs the user's immediate voice
attention). For everything else, prefer `notify`.

## Deep-link destination vocabulary

`--destination` accepts these values, each opening the matching iOS screen:

| Destination | Opens | Useful params |
| --- | --- | --- |
| `call` | Active call tab | `room_name` |
| `dispatch` | Dispatcher thread | — |
| `threads` | Coding threads (a specific thread) | `thread_id` |
| `sync_conflicts` | Sync conflicts | — |
| `approvals` | Approval requests | — |
| `reports` | Reports (a specific report) | `report_id` |
| `diff` | Diff viewer | `project_path`, `ref` |
| `account` | Settings / account | — |
| `voice_test` | LiveKit voice test | — |

The same vocabulary backs `openbase-app://route/<destination>?<params>` deep
links (for example `openbase-app://route/threads?thread_id=abc123`).

## Escalation guidance

- Default to `user say` for progress updates during work — it reaches the user
  on the call or via push automatically.
- Use `user notify` to point the user at a specific screen (a thread, a report,
  an approval) when they are not necessarily on a call.
- Use `user call` only to actually get the user on a live voice call.

## Notes

- Push and call delivery require the user to have opened and signed into the
  Openbase iOS app at least once so the device and its push tokens are
  registered with Openbase Cloud.
- If no iPhone is registered, `notify` and `call` report an error explaining the
  phone must open the app once. `user say` still speaks into an active call.
- This is distinct from `ios-app-control` (the `openbase-coder user ios ...`
  commands), which only works when the app is already foregrounded and connected
  to the local server. Push and call work even when the app is closed.

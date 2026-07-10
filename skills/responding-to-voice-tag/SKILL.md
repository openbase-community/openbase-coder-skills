---
name: responding-to-voice-tag
description: >-
  Use this skill whenever user input arrives wrapped in <voice> tags. The tags
  mean the text is a live speech transcription from an Openbase Coder voice
  session and your final response will be spoken aloud, so respond in a
  voice-appropriate way. Applies to every agent — dispatcher and Super Agents
  alike, on any coding backend.
version: 0.1.0
---

# Responding To Voice-Tagged Input

Input wrapped in `<voice>` tags is a live speech transcription from an
Openbase Coder voice session:

```
<voice>can you check whether the tests still pass</voice>
```

The tags are transport metadata added by the voice pipeline — the user did
not type them and never sees them. Everything between the tags is what the
user said, transcribed. Your final message for a voice-tagged turn is read
aloud by text-to-speech.

## How To Respond

- Keep final spoken responses short and directly useful.
- Write for the ear, not the eye: brief plain prose. Avoid bulleted or
  itemized lists — text-to-speech reads repeated item markers badly. When a
  list is genuinely clearer, use a short numbered list instead of bullets.
- Do not read code, logs, stack traces, JSON, diffs, thread IDs, or long file
  paths aloud unless the user explicitly asks. When code or logs matter,
  summarize their practical meaning in plain English.
- Speech-to-text mishears things. If a transcription is unclear or a heard
  name/term could plausibly be several different things, ask the user to
  confirm the intended request before acting on it.
- Never echo the `<voice>` tags back, and do not mention the tags or the
  transcription mechanics to the user.

## Voice Session Mechanics

- Only the user's speech is tagged. Untagged input in the same thread is
  ordinary text input and gets a normal (text-appropriate) response.
- Doing work is unchanged: edit files, run commands, and format code exactly
  as you normally would. Voice style applies to what you say back, not to
  the work itself.
- When the user asks to return to dispatch, or you need to hand the voice
  session back to dispatch, run:

  ```bash
  openbase-coder exit-to-dispatch
  ```

- Do not assume dispatcher responsibilities (routing, delegation policy, or
  Super Agents coordination rules) just because a turn arrived over voice;
  those come from your own role instructions.

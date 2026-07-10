---
name: openbase-onboarding
description: >-
  Use this skill when onboarding a user to Openbase Coder: setting up
  recommended integrations for email, meeting notes, shared documents,
  calendar, personal messaging, screen sharing/computer control, and the
  GitHub CLI. Trigger this skill when a message notes that onboarding has
  not been read yet, or when the user asks to set up integrations or
  redo onboarding.
version: 0.2.0
---

# Openbase Onboarding

This skill walks a new user through connecting the integrations that make
Openbase Coder useful day to day. The conversation may happen over voice, so
keep it conversational: introduce one category at a time, ask a question, and
wait for the answer instead of reading long lists aloud.

## First: mark onboarding as read

Before anything else — even if the user declines onboarding or you complete
only part of it — create the marker file that records the onboarding skill
has been read:

```bash
touch ~/.openbase/onboarding-skill-read
```

This stops Openbase Coder from re-prompting onboarding on every future
message. Create it immediately upon reading this skill; do not wait for
onboarding to finish. If the user wants to continue onboarding later, they
(or you) can just ask for the `openbase-onboarding` skill by name.

## Ground rules for every integration

- **Prefer CLIs and skills over MCP servers.** Do not install or attach MCP
  servers; they bloat agent context and lack progressive disclosure. If a
  service only offers an MCP server, use `mcporter` to generate a local,
  deterministic CLI from it instead of connecting the MCP directly.
- **Install skills only from official sources** — the vendor's official CLI
  or the officially supported skill repositories listed below. Never install
  a skill from an unvetted third-party repository.
- **Ask before installing.** For each category, ask which tool or service the
  user prefers, confirm before installing anything, and respect a "skip".
- **Offer a test after each setup.** Once a tool is installed and
  authenticated, ask whether the user wants to try it out, and run a small,
  read-only test if they say yes (e.g. list recent emails, list upcoming
  events).
- **Install location for skills:** clone or place personal skills in
  `~/.agents/skills/<skill-name>` (each skill directory contains a
  `SKILL.md`). Openbase Coder auto-links skills from there into its agent
  homes when personal-skill auto-linking is enabled.
- Many setups require the user to complete an OAuth or login flow in a
  browser or terminal. Tell the user clearly when you need them to act, and
  wait.

## Onboarding flow

Work through the categories in order. For each: briefly say what it enables,
ask which tool the user prefers (or whether to skip), set it up, then offer a
test.

### 1. GitHub CLI

Ensure `gh` is installed and authenticated — many coding workflows depend on
it:

```bash
gh --version || brew install gh
gh auth status || gh auth login
```

`gh auth login` is interactive; guide the user through choosing GitHub.com
and their preferred auth method.

### 2. Session-ID commit hooks

`openbase-coder setup` installs a SessionStart hook
(`~/.openbase/hooks/inject-session-id.sh`) into both Openbase agent homes so
every agent knows its own thread/session ID and stamps commits with an
`Agent-Thread-Id` trailer that ties each commit back to the session that
produced it. No user choice is needed here — just verify it is in place:

```bash
test -x ~/.openbase/hooks/inject-session-id.sh \
  && grep -q inject-session-id ~/.openbase/claude_config/settings.json \
  && grep -qF '[[hooks.SessionStart]]' ~/.openbase/codex_home/config.toml \
  && echo ok
```

If anything is missing, re-run `openbase-coder setup` (it is idempotent) and
verify again. Briefly tell the user what the hook does; there is nothing for
them to configure.

### 3. Email

Ask which email provider the user relies on:

- **Gmail** (officially supported): install the
  [gmail-cli-skill](https://github.com/montaguegabe/gmail-cli-skill) into
  `~/.agents/skills`. It is a narrow CLI that enforces minimum OAuth scopes
  and gates sending behind explicit approval. Follow its README/SKILL.md for
  OAuth setup.
- **Other providers**: note the preference and skip rather than installing
  anything unofficial.

### 4. Meeting notes

Ask whether the user records meetings and with what:

- **Fathom** (officially supported): install the
  [fathom-skill](https://github.com/montaguegabe/fathom-skill), an
  mcporter-generated CLI wrapper around the upstream Fathom MCP server —
  OAuth/session state and generated code stay local.
- Other note-taking services: skip unless an official CLI exists.

### 5. Shared documents (Notion or Google Drive)

Ask where the user's shared documents live:

- **Notion**: use the
  [official Notion CLI](https://developers.notion.com/cli/get-started/overview)
  (`ntn`) — it already provides a stable, supported surface for pages, data
  sources, files, and API calls. Install it and run `ntn login`.
- **Google Drive / Google Workspace**: use the official
  [Google Workspace CLI](https://github.com/googleworkspace/cli) (`gws`).
- Both is fine — set up each one the user wants.

### 6. Calendar management

Ask what calendar the user keeps:

- **Google Calendar**: covered by the official
  [Google Workspace CLI](https://github.com/googleworkspace/cli) (`gws`) —
  reuse the auth from the shared-documents step if already done.
- Other calendars: skip unless an official CLI exists.

### 7. Personal communication (optional)

This category is optional — ask whether the user wants agents to be able to
read and (with approval) send personal messages:

- **iMessage** (officially supported, macOS only): install the
  [imessage-skill](https://github.com/montaguegabe/imessage-skill). It needs
  Full Disk Access for the host terminal/agent process, which the user must
  grant in System Settings; it limits message text to approved conversations.
- **WhatsApp** (officially supported): install the
  [whatsapp-skill](https://github.com/montaguegabe/whatsapp-skill). It keeps
  sending behind approved contacts and explicit approval.
- The user can choose either, both, or neither.

### 8. Screen sharing / computer control

Set up desktop computer use last, and only when the user is physically at the
Mac: macOS requires them to grant Screen Recording and Accessibility
permissions in System Settings, which cannot be done remotely.

- Follow the bundled `openbase-computer-use` skill for the correct
  environment-specific path and commands.
- Ask the user to confirm they are at the computer before starting. If they
  are not, note it as unfinished and tell them any agent can pick this up
  later via the `openbase-onboarding` or `openbase-computer-use` skill.
- After permissions are granted, offer a quick test (e.g. take a screenshot
  or start and stop a screen share).

## Wrapping up

Summarize what was set up, what was skipped, and anything left unfinished
(for example computer-use permissions). Remind the user they can resume
anytime by asking for onboarding.

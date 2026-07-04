---
name: openbase-product-knowledge
description: >-
  Use this skill when answering questions about the Openbase Coder product:
  what it is, what the desktop app, iOS app, web console, or Openbase Cloud
  can do, how the pieces connect, or where a feature or setting lives. Routes
  the agent to the correct page of the product documentation instead of
  guessing from code.
version: 0.1.0
---

# Openbase Product Knowledge

Openbase Coder's documentation covers the whole product — not just the CLI.
When a user (or the dispatcher, or a voice caller) asks what Openbase Coder
can do, how to do something in one of the apps, or where a setting lives,
answer from the docs rather than from memory or ad hoc code reading.

## When To Use

Use this skill when:

- the user asks what Openbase Coder is or what it can do
- the question is about the Mac desktop app, the iOS app, the web console,
  or app.openbase.cloud ("can I do X from my phone?", "where do I change
  the voice?", "how do I approve an agent request?")
- the question is about setup, pairing an iPhone, Tailscale, voice calls,
  updating, uninstalling, or troubleshooting connectivity
- an agent needs to explain or verify user-visible product behavior before
  changing it

## Where The Docs Live

The docs source of truth is the `cli` repo's `docs/` directory
(`cli/docs/` in the `openbase-coder-workspace` multi workspace). The same
content is published at `https://docs.openbase.cloud` (page URLs follow the
file layout: `docs/desktop-app.md` → `/desktop-app/`,
`docs/commands/setup.md` → `/commands/setup/`).

Resolution order:

1. If working inside the workspace or a `openbase-coder` checkout, read the
   Markdown under `cli/docs/` directly.
2. Otherwise fetch the page from `https://docs.openbase.cloud`.

Read the specific page for the topic; do not answer product questions purely
from source code when a docs page exists. Code is the fallback for details
the docs do not cover.

## Product Mental Model

One local runtime, several faces:

- **Desktop app (macOS Electron)** — bundles and activates the CLI runtime,
  runs guided setup, and hosts the dashboard (projects, threads, reports,
  dispatch, approvals, routines, skills, templates, status, settings) plus
  screen sharing and auto-update.
- **iOS app** — voice calls with the dispatcher and Super Agents, threads,
  approvals, reports, diffs, and phone-side settings, connected to the Mac
  over Tailscale (or to a Cloud DevSpace).
- **Web console** — the same dashboard served by the local runtime in any
  browser; also embedded by the desktop and iOS apps.
- **app.openbase.cloud** — the Openbase Cloud account: sign-in/OAuth, device
  onboarding for iPhone pairing, subscription, and Cloud DevSpace launch.
  Its deployments tooling is out of scope for product answers.
- **`openbase-coder` CLI** — the local Django API + WebSocket server,
  LiveKit voice services, and service management underneath everything.

Focus answers on what the user actually sees — the desktop app and iOS app
first, then the console and cloud — and mention the CLI only as the
underlying mechanism or for terminal-preferring users.

## Docs Routing

Start pages:

- Product overview and "which page do I need" → `docs/index.md`
- Desktop app (setup flow, every dashboard page, settings, screen sharing,
  auto-update, deep links) → `docs/desktop-app.md`
- iOS app (onboarding, every tab, voice calls, Action Button mute, push
  notifications, connection details) → `docs/ios-tabs.md`
- Web console access and app.openbase.cloud → `docs/console.md`

By topic:

- Installing / first-time setup → `docs/getting-started.md`, and
  `docs/manual-installation.md` for the terminal-driven desktop setup
- Download links (Mac, iOS, Android, CLI) → `docs/downloads.md`
- Voice call routing, transferring to Super Agents, speaking announcements
  → `docs/voice-routing.md`
- Cloud DevSpace (Linux sandbox instead of a Mac) → `docs/cloud-devspace.md`
- Fully local audio/models (privacy mode) → `docs/local-only.md`
- iPhone won't connect, LiveKit call timeouts, voice route errors
  → `docs/troubleshooting.md`
- Updating → `docs/commands/self-update.md` (CLI) and the Auto-Update
  section of `docs/desktop-app.md` (apps)
- Uninstalling → `docs/uninstall.md`
- CLI commands → `docs/commands/index.md` and per-command pages
- Environment variables, auth modes → `docs/configuration.md`
- What file/directory is what → `docs/files-and-paths.md`
- Plugins → `docs/commands/plugins.md`, `docs/plugins/index.md`
- Muted-call music / Brain Readiness score
  → `docs/plugins/brain-score-concurrency.md`

Workspace-level guides (developer-facing, in `openbase-coder-workspace`):
`DEV_RUNBOOK.md` (dev install/test flow), `AUTO_UPDATE.md` (release/update
contracts), `GLOSSARY.md` (terminology), `specs/` (cross-repo feature
specs). Prefer these over `cli/docs/` only for development and release
questions.

## Answering Style

- Name the surface in the answer ("in the desktop app under
  **Settings → Coding Backend**…", "on iPhone, the Approvals tab…").
- When a capability exists on several surfaces, say so briefly — most
  dashboard pages exist in both the desktop app and the console, and many
  have an iOS counterpart; the docs pages carry "On iPhone" / "In the apps"
  notes for exactly this.
- If the docs are missing or contradict observed behavior, verify against
  the code, then fix the docs page in `cli/docs/` (keeping its
  cross-surface notes) rather than leaving the answer only in chat.

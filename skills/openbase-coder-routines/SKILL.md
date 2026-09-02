---
name: openbase-coder-routines
description: >-
  Use this skill when working with openbase-coder routines commands,
  routine scheduling, routine console behavior, or deciding whether routines
  are Codex app-server native.
version: 0.1.0
---

# Openbase Coder Routines

Use Openbase Coder routines from the CLI and console without exposing routine
operations as Super Agents MCP tools.

## When To Use

Use this skill when:

- the user asks how to create, list, update, run, or delete routines
- the user asks whether routines are a first-class Codex app-server concept
- the user asks for routine scheduler, launchd, or run-on-time behavior
- the task touches the Routines console page or `/api/routines/` endpoints
- the task could accidentally reintroduce routine MCP tools

## Core Model

- Routines are an Openbase Coder/Super Agents wrapper concept, not a native
  Codex app-server concept.
- Routines have two first-class kinds:
  - `agent` routines start or queue a Codex app-server/Super Agents turn.
  - `command` routines run a normal local shell command and do not launch an AI
    agent by themselves.
- Codex app-server owns threads and turns. Only `agent` routine execution calls
  the same app-server `turn/start` path used by normal Super Agents turns.
- Routine state is persisted in the local Super Agents state file through
  `super_agents.app_server_client.CodexAppServerClient`.
- Routine control should be exposed through `openbase-coder routines ...`, the
  local console API, and the Routines console page.
- Do not add `super_agents_create_routine`, `super_agents_list_routines`,
  `super_agents_read_routine`, `super_agents_update_routine`,
  `super_agents_delete_routine`, or `super_agents_run_due_routines` back to the
  MCP tool surface.

## Main Commands

Assume `openbase-coder` is installed on `PATH`. Use the CLI as the canonical
user-facing command surface:

```bash
openbase-coder routines list
openbase-coder routines show ROUTINE_NAME
openbase-coder routines create ROUTINE_NAME --prompt "Inspect project health." --time 09:00
openbase-coder routines create ROUTINE_NAME --kind command --command "make check" --time 09:00
openbase-coder routines create poller --prompt "Poll for ready work." --interval-seconds 60
openbase-coder routines update ROUTINE_NAME --disable
openbase-coder routines update ROUTINE_NAME --enable
openbase-coder routines run-due
openbase-coder routines run-due --name ROUTINE_NAME --force
openbase-coder routines delete ROUTINE_NAME
```

`openbase-coder loops ...` is an alias for the same command group and is the
preferred spelling in user-facing guidance.

Routines can also run on external events through webhook triggers
(`add-webhook-trigger`, `remove-trigger`, `emit`, the
`/api/routines/<name>/triggers/` endpoints, and the public
`/api/hooks/t/<token>/` ingest endpoint). Use the `openbase-loops` skill for
trigger creation, provider wiring, and the sender-allowlist security rules; do
not expose trigger operations as Super Agents MCP tools either.

Routine creation supports these fields where needed:

```bash
openbase-coder routines create daily-check \
  --kind agent \
  --prompt "Inspect project health." \
  --time 09:00 \
  --timezone America/New_York \
  --target-name "workspace-health" \
  --cwd PROJECT_PATH \
  --mode default \
  --approval-policy never \
  --sandbox-type dangerFullAccess
```

Interval routines are first-class and use `scheduleType=interval` plus
`intervalSeconds`. Passing `--interval-seconds` to the CLI implies an interval
routine:

```bash
openbase-coder routines create notion-prioritized-poller \
  --prompt "Check Notion for prioritized work and dispatch ready cards." \
  --interval-seconds 60 \
  --cwd PROJECT_PATH \
  --mode default
```

Use `--thread-id` instead of `--target-name` when the routine must target one
specific existing Codex app-server thread.

Command routines use `--kind command` and `--command`. They may keep a prompt as
handoff context, but the command is the routine start action:

```bash
openbase-coder routines create workspace-report-routine \
  --kind command \
  --command "generate-workspace-report --workspace /path/to/workspace" \
  --interval-seconds 300 \
  --cwd /path/to/workspace
```

When a command routine feeds a later agent step, keep discovery deterministic:
the command should enumerate and bound the work items itself (emitting a
precise list or JSON payload), so the agent consumes bounded requests instead
of rediscovering state. If the command hands off turns programmatically, it
should use the Super Agents Python client/library rather than shelling out to
`openbase-coder`.

## Console And API

The console page is:

```text
/dashboard/routines
```

The local API endpoints are:

```text
GET    /api/routines/
POST   /api/routines/
GET    /api/routines/<name>/
PATCH  /api/routines/<name>/
DELETE /api/routines/<name>/
POST   /api/routines/run-due/
```

The API should delegate to the Super Agents Python client library. It should not
shell out to the CLI from inside Django.

## Source Locations

In the `openbase-coder-workspace` multi workspace, the main files are:

```text
super-agents/src/super_agents/app_server_client.py
super-agents/src/super_agents/state.py
super-agents/src/super_agents/mcp_server.py
cli/openbase_coder_cli/cli/routines.py
cli/openbase_coder_cli/thread_sync/session_manager.py
cli/openbase_coder_cli/openbase_coder_cli_app/views.py
cli/openbase_coder_cli/openbase_coder_cli_app/urls.py
coder-react/src/pages/Routines.tsx
coder-react/src/App.tsx
coder-react/src/components/layouts/ExampleLayout.tsx
```

## Scheduler Guidance

For reliable run-on-time behavior, prefer a dedicated local scheduler process
over tying execution to MCP server lifetime:

1. Run `openbase-coder routines run-loop --interval 60` under launchd.
2. Keep `openbase-coder routines run-due` idempotent by checking `lastRunDate`
   for daily routines and `lastRunAt` for interval routines.
3. Add file locking around state updates before relying on concurrent scheduler
   and console/manual runs.
4. Persist scheduler health, such as last tick and last error, if the console
   needs operational status.
5. Keep manual `run-due --name ROUTINE_NAME --force` for debugging and one-off
   runs.

Do not model routines as a Codex app-server first-class feature unless the
app-server protocol adds native routine storage or scheduling.

## Validation

After changing routine code, run the focused checks from the relevant subrepos:

```bash
cd super-agents
uv run pytest tests/test_app_server_client.py
uv run ruff check src/super_agents/mcp_server.py tests/test_app_server_client.py

cd ../cli
uv run pytest tests/test_session_manager.py tests/test_routines_cli.py
uv run ruff check openbase_coder_cli/cli/routines.py openbase_coder_cli/thread_sync/session_manager.py openbase_coder_cli/openbase_coder_cli_app/views.py openbase_coder_cli/openbase_coder_cli_app/urls.py tests/test_session_manager.py tests/test_routines_cli.py

cd ../console
pnpm exec tsc -p tsconfig.json --noEmit
pnpm build
```

## Notes

- If a user asks for a "routine page", use the existing Routines page instead
  of adding routine controls to Launchctl.
- If a user asks "do routines require Codex app-server support?", answer that
  command routines do not; agent routines require Codex app-server when
  launching the actual thread/turn.
- Keep routine behavior backward compatible with the persisted Super Agents
  state shape.

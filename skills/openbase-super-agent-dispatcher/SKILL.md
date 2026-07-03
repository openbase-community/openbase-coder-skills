---
name: openbase-super-agent-dispatcher
description: >-
  Use this skill when dispatching, starting, continuing, steering, transferring,
  or otherwise managing Openbase Super Agents from a dispatcher or from another
  Super Agent.
version: 0.1.0
---

# Openbase Super Agent Dispatcher

Use the Openbase Super Agents MCP path for Super Agent delegation. Generic
subagents are not Openbase Super Agents and must not be substituted for them.

## When To Use

Use this skill when:

- the user asks for a Super Agent
- the user asks you to start, continue, steer, or check a Super Agents thread
- the user asks to transfer the active voice session to or from a Super Agent
- the task is multi-file implementation work or a detailed investigation that
  should be delegated
- a Super Agent is explicitly being asked to act as a dispatcher

For multi-file implementation work, default to using a Super Agent by starting a
new thread or continuing a related existing one. If the user asks you to do the
multi-file edits yourself, confirm that they want you to personally make those
edits before proceeding. For detailed investigations, similarly default to
using a Super Agent.

## Starting Threads

- Delegate promptly, then get back to the user immediately after the turn is
  started. Do not spend extra time refining the prompt in silence unless the
  request is genuinely ambiguous or unsafe.
- Choose the final thread `name` first. Derive the required `agentName` from
  that exact thread name with:

```bash
openbase-coder super-agent-name "<thread name>" --json
```

- Pass the returned `agent_name` as `agentName` in `super_agents_start` and in
  any first `super_agents_start_turn` call that immediately follows.
- Keep the thread `name` task-focused, such as `implement-my-feature`. Keep
  `agentName` to the returned person/voice name, such as `Carl` or `Dottie`.
- Start the thread with `sandbox: "danger-full-access"`.
- Start the turn with `sandboxType: "dangerFullAccess"`.
- If the Super Agent is expected to run in the background, instruct it to
  announce completion with:

```bash
openbase-coder user say "<agent name>" "<message>"
```

Use the derived `agentName` from `openbase-coder super-agent-name` and the
Super Agents MCP calls as the speaking agent name.

## Prompting

- Pass on the user's request directly without adding too much extra detail.
- Add detail only when it clarifies paths, repo/workspace context, safety
  constraints, or parts of the request that are not easy to speak clearly over
  voice.
- Do not add a task-specific "before starting work, introduce yourself"
  instruction when standard Super Agent instructions are already included.
  Those standard instructions own the one-time introduction.
- If an introduction reminder is unavoidable for a prompt that might not include
  standard Super Agent instructions, say that any introduction reminder is
  idempotent and must be satisfied by exactly one canonical command:

```bash
openbase-coder user say "<agent name>" "Hey there, I'm <agent name>."
```

Use the same `agentName` passed to the Super Agents MCP call.

## After Starting

After starting a Super Agents turn, immediately tell the user that the agent has
started and briefly summarize what the agent is doing. Do not ask when the user
wants progress checked; the user will ask. Do not silently wait for the turn to
finish unless the user explicitly asks you to wait.

Do not assume that threads or Super Agents are still running without checking
unless you just launched them.

## Continuing And Steering

- When sending work to an actively working Super Agents thread, prefer queuing a
  new follow-up turn for new features or distinct work items.
- Use steering for clarifications, follow-ups, or modifications to an existing
  active task.
- Push back when the user tries to start multiple Super Agents in the same repo
  or workspace if the tasks seem likely to collide, edit overlapping files,
  fight over branches, or confuse ownership of the same implementation. Explain
  the collision risk briefly and suggest reusing one agent, sequencing the work,
  or creating separate worktrees before proceeding.

## Projects And Workspaces

When using the Super Agents MCP for a new project, create or choose a dedicated
project folder first, then start the Super Agent session with that folder as the
working directory using the MCP.

## Plan Mode

- When a plan-mode question appears, present the available choices to the user
  one at a time and wait for the user's explicit answer.
- Do not answer Super Agents plan-mode questions on the user's behalf unless the
  user explicitly asks you to choose.
- When multiple plan-mode questions appear, ask the user the questions one at a
  time. Wait for the user's explicit answer to each question before presenting
  the next question.
- After a plan-mode turn completes, explain that implementation requires a new
  default-mode turn on the same thread and working directory. To implement,
  start a new Super Agents turn in default mode with a prompt like
  `Implement the plan.` Do not assume plan mode will automatically ask to
  implement.

## Voice Transfer

When the user asks to transfer the active voice session to a Super Agent, use
the Openbase Coder CLI instead of asking for confirmation. Transferring is
private voice-session routing, not a destructive command or public publishing
command.

To transfer to a named Super Agent:

```bash
openbase-coder user transfer-to-agent "<agent name>"
```

To transfer by thread ID:

```bash
openbase-coder user transfer-to-thread "<thread id>"
```

When creating or referring to a Super Agent by thread name, derive the speaking
agent name first with:

```bash
openbase-coder super-agent-name "<thread name>" --json
```

Then pass the returned `agent_name` to transfer commands and Super Agents MCP
calls.

## Persistent Memories

When the user says "always remember" or otherwise asks you to permanently
remember an instruction, treat that as a request to update one of the following
instruction files that form agent memory:

- `~/.openbase/codex_home/AGENTS.md`: Codex general instructions.
- `~/.openbase/claude_config/CLAUDE.md`: Claude Code general instructions.
- `~/.openbase/instructions/VOICE_INSTRUCTIONS.md`: Voice-channel
  instructions.
- `~/.openbase/instructions/DISPATCHER_INSTRUCTIONS.md`: Super Agent
  dispatching and management instructions.

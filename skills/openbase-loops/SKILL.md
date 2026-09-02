---
name: openbase-loops
description: >-
  Use this skill when connecting external event sources (GitHub webhooks,
  generic webhooks, local events) to Openbase Coder loops, adding or removing
  loop triggers, emitting test events, or explaining the loop/trigger/event
  model. Openbase ships no provider connector catalog: agents wire providers
  themselves with this skill.
version: 0.1.0
---

# Openbase Loops And Triggers

A loop is what should happen repeatedly; triggers are when it happens. Openbase
deliberately maintains no per-provider connector catalog. Instead, this skill
teaches the agent to wire any provider to a loop using generic webhook
triggers.

## Vocabulary

- **Loop**: a persisted routine (`agent` or `command` kind). "Loop" is the
  user-facing name for a routine; the storage and API keep the `routine` name.
- **Trigger**: when the loop should run. Every loop has an implicit schedule
  trigger (`daily` or `interval`) plus optional webhook triggers.
- **Event**: one delivery that matched a trigger.
- **Run**: one execution of the loop.

## Creating A Webhook Trigger

```bash
openbase-coder loops add-webhook-trigger LOOP_NAME \
  --description "Pull request comments on owner/repo" \
  --sender-path sender.id \
  --allow-sender 12345 \
  --filter comment.body startsWith /openbase \
  --hmac-secret "$(openssl rand -hex 32)"
```

The command prints the trigger, including its capability `token` and
`ingestPath` (`/api/hooks/t/<token>/`). The token is the URL credential — treat
it like a secret. To rotate, remove the trigger and add a new one.

- `--filter PATH OP VALUE` is repeatable. Ops: `equals`, `notEquals`,
  `contains`, `startsWith`, `endsWith`, `exists`, `regex`. Paths are dot paths
  into the JSON payload (`comment.body`, `sender.id`).
- `--hmac-secret` enables SHA-256 HMAC verification of the raw body. The
  signature header defaults to `X-Hub-Signature-256` (GitHub's convention);
  override with `--hmac-header` for other providers.
- Remove with `openbase-coder loops remove-trigger LOOP_NAME TRIGGER_ID`.
- The console equivalent lives on the loop detail page (`/dashboard/loops`).

## Sender Allowlists Are Mandatory For Agent Loops

An externally reachable trigger that starts an agent turn is a prompt-injection
port: whoever can produce the event authors the prompt context. Therefore:

- Agent loops REQUIRE `--sender-path` and at least one `--allow-sender`;
  creation fails without them, and delivery denies external events that do not
  match. Never work around this by switching a loop to `command` kind to skip
  the check when the command hands off to an agent.
- The sender value is read from the signature-verified payload at
  `senderPath` and compared as a string. For GitHub use `sender.id` (numeric,
  immutable) rather than `sender.login`. Get the user's own id with
  `gh api user --jq .id`.

## Wiring GitHub (No GitHub App Needed)

Use the user's own repo webhook — requires repo admin:

```bash
SECRET=$(openssl rand -hex 32)
openbase-coder loops add-webhook-trigger pr-feedback --cloud \
  --description "PR comments on OWNER/REPO" \
  --sender-path sender.id \
  --allow-sender "$(gh api user --jq .id)" \
  --filter comment.body startsWith /openbase \
  --hmac-secret "$SECRET"
# Take "providerUrl" from the output (requires --cloud), then:
gh api repos/OWNER/REPO/hooks -f name=web \
  -F 'events[]=issue_comment' -F 'events[]=pull_request_review_comment' \
  -F config[url]="PROVIDER_URL" \
  -F config[content_type]=json -F config[secret]="$SECRET"
```

Event ids come from `X-GitHub-Delivery`, so provider redeliveries deduplicate
automatically.

## Reachability: Prefer The Cloud Relay

Pass `--cloud` to `add-webhook-trigger` to get a publicly reachable URL with
no tunnel or port publishing:

```bash
openbase-coder loops add-webhook-trigger LOOP_NAME --cloud ...
```

This creates an Openbase Cloud relay endpoint and prints `providerUrl` — the
URL to paste into the provider. Cloud verifies nothing provider-specific: it
stores the raw delivery durably at that capability URL, acks the provider
within its timeout, and this machine's `sync-workers` service polls pending
events (default every 30s, `OPENBASE_CLOUD_WEBHOOK_POLL_INTERVAL`), hands each
one to the local trigger pipeline (same HMAC verification, filters, sender
allowlist, and dedup as a direct delivery), and acks what it handled. Events
for relay endpoints that do not match a local trigger are left pending for the
user's other devices. A sleeping machine catches up on its next poll; cloud
retention is 14 days for unclaimed events.

Without `--cloud`, the local ingest endpoint still works for callers that can
reach this machine:

- For devices on the user's tailnet, use the tailnet address of the machine.
- To expose the endpoint publicly without the relay, use the
  `openbase-service-publishing` skill to publish the local API port.
- For local automation and testing, skip HTTP entirely:
  `openbase-coder loops emit LOOP_NAME --data '{"note": "manual"}'`.

## Delivery Semantics

- Duplicate event ids per trigger are dropped (`status: duplicate`).
- Non-matching payloads return `status: filtered`; unauthorized senders return
  `status: unauthorized_sender`. Both still update trigger event stats.
- Event-driven runs never touch `lastRunDate`/`lastRunAt`, so they cannot
  displace or delay the loop's own schedule.
- Agent loops receive the event appended to the prompt as a
  "Triggering event" section. Command loops receive it as the
  `SUPER_AGENTS_EVENT_JSON` environment variable.

## Hard Rules

- Do not build or propose a per-provider connector registry in Openbase core;
  provider knowledge belongs in skills like this one.
- Never store provider payload processing secrets in tracked files; pass HMAC
  secrets through the CLI/console at trigger creation.
- Personal/user-specific loops must not be baked into Openbase source; create
  them through this generic surface.

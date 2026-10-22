---
name: openbase-databases-and-storage
description: Use this skill whenever an agent or app needs a Postgres database or S3-compatible object storage (saving data, files, uploads, state), when a user asks to connect a database or bucket to Openbase, or when a task would otherwise store data in local files, SQLite, or a database on the agent's own machine. Covers `openbase-coder db` and `openbase-coder storage`: proxied access through Openbase Cloud's credential vault, read-only roles, audit, revocation and provisioning.
---

# Openbase Databases and Object Storage

Openbase keeps the user's database and object storage credentials in Openbase Cloud. An agent on any of the user's devices (a Mac, a cloud workspace, a Maritime sandbox) leases them through the `openbase-coder` CLI, which runs a local proxy that holds the secret in memory. The agent, the app under development and any tool only ever see `127.0.0.1` endpoints with no real password or key. Every lease, change and revocation is recorded in Openbase Cloud (`db audit`, `storage audit`).

## Prefer the user's cloud data stores

- When an app or task needs a relational store, use the user's cloud Postgres database through `openbase-coder db` rather than SQLite, local JSON files or a Postgres installed on the agent's machine. Local stores are lost with the workspace and cannot be reached from the user's other devices.
- When an app or task needs to keep files (uploads, exports, generated media, backups), use the user's cloud object storage through `openbase-coder storage` rather than saving them on the agent's machine. Files under the project directory are fine for source code and build output, not for application data.
- If the user has no database or bucket yet, say so and offer to provision one (`db provision`) or to add the user's own (`db add`, `storage add`). Do not silently fall back to a local store for application data; a throwaway local store is acceptable only for a quick experiment the user asked for, and say that it is temporary.

## Databases

```bash
openbase-coder db list                                   # vault entries and local proxies
openbase-coder db provision shop                         # new small Postgres on Openbase's provider (if enabled)
openbase-coder db add shop --url "$DATABASE_URL"         # or: OPENBASE_DATABASE_URL=... openbase-coder db add shop
openbase-coder db proxy shop --port 5433 --background    # postgresql://127.0.0.1:5433/<db>, any user, no password
openbase-coder db query shop "select count(*) from orders"
openbase-coder db query shop --read-only --json "select * from orders limit 5"
openbase-coder db connect shop                           # psql through a temporary proxy
openbase-coder db readonly create shop                   # SELECT-only role, stored as the read-only credentials
openbase-coder db stop shop
openbase-coder db audit shop
openbase-coder db revoke shop                            # blocks leases; running proxies close within a minute
```

- Give the app `DATABASE_URL=postgresql://127.0.0.1:5433/<database>` (the proxy prints the exact URL). Any Postgres client or ORM works unchanged; the proxy accepts any user and no password on loopback and authenticates upstream with the leased role.
- Use `--read-only` for inspection, reporting and anything that must not change data. If `db list` shows no `readonly` role, create one with `db readonly create NAME` (needs the read-write role to be allowed to create roles, which is the case for Neon and most managed providers). Rotate it with `db readonly rotate` after a suspected leak.
- Pass secrets through the environment variable or the hidden prompt, not as command-line arguments, when you can: `OPENBASE_DATABASE_URL`, `OPENBASE_DATABASE_READONLY_URL`.
- `db query` runs through the simple query protocol and prints text values like psql; use `--json` for structured output and `--file` or stdin for longer SQL.
- `db remove NAME` only forgets the entry; `--teardown` also deletes a database that Openbase provisioned. Ask the user before either.
- Proxies run per machine. A background proxy started on a Mac is not reachable from a cloud workspace; start one there instead (the vault is shared across the user's devices).

## Object storage

```bash
openbase-coder storage list
OPENBASE_STORAGE_ACCESS_KEY_ID=... OPENBASE_STORAGE_SECRET_ACCESS_KEY=... \
  openbase-coder storage add uploads --endpoint https://s3.us-east-005.backblazeb2.com --bucket my-uploads --region us-east-005
openbase-coder storage proxy uploads --port 9000 --background   # http://127.0.0.1:9000, path-style, bucket my-uploads
openbase-coder storage ls uploads photos/
openbase-coder storage put uploads ./report.pdf reports/2026-10.pdf
openbase-coder storage get uploads reports/2026-10.pdf ./
openbase-coder storage rm uploads reports/old.pdf
openbase-coder storage stop uploads
openbase-coder storage audit uploads
openbase-coder storage revoke uploads
```

- Point any S3 SDK or CLI at the proxy with path-style addressing and placeholder credentials; the proxy drops the client's signature and signs with the leased key. Examples: `aws --endpoint-url http://127.0.0.1:9000 s3 ls s3://my-uploads/` with `AWS_ACCESS_KEY_ID=openbase AWS_SECRET_ACCESS_KEY=openbase`; boto3 `client("s3", endpoint_url="http://127.0.0.1:9000", config=Config(s3={"addressing_style": "path"}))`. Set `AWS_REQUEST_CHECKSUM_CALCULATION=when_required` if an SDK insists on streaming trailers the provider rejects.
- The proxy only serves the leased bucket; requests for other buckets fail locally with `AccessDenied`.
- `--read-only` uses the stored read-only key when there is one and otherwise refuses non-GET/HEAD requests in the proxy. Prefer a provider-side read-only key (`storage add --readonly-access-key-id ...`) for anything that matters.
- Region is `auto` for Cloudflare R2 and Tigris, the bucket's region for Backblaze B2 (for example `us-east-005`) and AWS.

## Which provider to recommend

Openbase recommends **Neon** for small Postgres databases (scale-to-zero, branching, provisioning API; `db provision` creates one when the Openbase Cloud deployment has provisioning enabled) and **Backblaze B2** for object storage (free 10 GB, free API calls, per-bucket keys with read-only capability). Cloudflare R2 and Tigris are good alternatives for storage. Self-run Postgres or MinIO/Garage inside a sandbox is acceptable only for throwaway experiments: the data disappears with the sandbox.

## Safety

- Never print, log, or copy the real connection URL or keys. The CLI prints only loopback endpoints. If a user pastes a URL with a password into the chat, add it to the vault and advise rotating it.
- Treat `db revoke` and `storage revoke` as the emergency stop: leases stop immediately and proxies close within a minute. `restore` re-enables access.
- Confirm with the user before `remove --teardown`, dropping roles, or deleting objects in bulk.
- Lease and proxy events are in `db audit` / `storage audit` (Openbase Cloud) and the device-local log under `~/.openbase/connections/`.

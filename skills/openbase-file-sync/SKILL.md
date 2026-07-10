---
name: openbase-file-sync
description: >-
  Use this skill when setting up or diagnosing file sync between a user's
  computers with Openbase Coder code sync or Syncthing: pairing a laptop with
  a desktop/Mac mini or DevSpace, SSH key access between devices, keeping
  sync on the Tailscale tailnet, Syncthing GUI credentials, sync conflicts,
  or why git repos must never sync their .git directories.
version: 0.1.0
---

# Openbase File Sync

Keep a user's second machine a live replica of their working files — dirty
changes just appear on the other device, with no commits required — without
ever letting file sync corrupt git state.

## When To Use

Use this skill when:

- the user wants files synced between two of their machines (laptop ⇄
  desktop/Mac mini, laptop ⇄ Cloud DevSpace)
- setting up or checking `openbase-coder sync` (code sync)
- diagnosing sync conflicts, `*.sync-conflict-*` files, out-of-sync folders,
  or "directory has been deleted on a remote device" errors
- reviewing a user-managed Syncthing installation that touches code
- the user asks about SSH access between their devices for sync or
  administration

## Hard Rules

1. **`.git` (and any VCS metadata: `.jj`, `.hg`) must NEVER travel over file
   sync.** Syncthing moves files independently with no transactional
   grouping; syncing a live `.git` tears refs/index state and silently
   corrupts commits. Git state moves only through git's own transports
   (code sync's reconciler does this automatically).
2. **Ignore files do not sync.** Syncthing keeps `.stignore` machine-local,
   and a symlinked include target outside the synced folder never
   propagates. Any safety-critical ignore pattern must be applied and
   verified **on every device separately** — never assume the other machine
   inherited it.
3. **Verify ignores against the running instance**, not the file: `GET
   /rest/db/ignores?folder=<id>` must show the expanded patterns (e.g.
   `(?d).git`). If a `.stignore` edit is not reflected, trigger `POST
   /rest/db/scan?folder=<id>`.
4. Secrets (`.env`, keys) inside synced project folders sync **on purpose**
   in code sync — that is a feature git cannot provide. Do not "fix" it.

## Prefer Product Code Sync

`openbase-coder sync` manages an isolated Syncthing instance end to end and
is the right answer for new setups (the user-managed path below is for
diagnosing setups the user built themselves).

- `openbase-coder sync enable` — installs a pinned Syncthing (checksum
  verified), renders config, writes managed `.stignore` files, starts the
  `code-sync` service. Requires two or more non-phone devices with
  Tailscale registered to the account (`--force` overrides).
- `openbase-coder sync add <path>` / `remove <path>` — folders are
  identified by home-relative path, so every device mounts the same folder
  at `$HOME/<relpath>`. Only paths under `~` are eligible; `~/.openbase`
  never syncs.
- `openbase-coder sync status` / `conflicts` / `resolve <id>
  --action keep_local|use_remote` — sync health and conflict handling.
- `openbase-coder sync reconcile` — one git-state reconcile pass (also runs
  automatically every minute via the routines service): fast-forwards a
  branch only when the peer's commit tree already matches the local working
  tree and the local head is an ancestor; anything diverged becomes a
  conflict record instead.
- `openbase-coder doctor` — includes code-sync checks (managed ignores
  intact, service running, version history size) and warns if a
  user-managed Syncthing is running without a `.git` ignore.

Facts that matter when debugging it:

- Managed instance: config at `~/.openbase/code-sync/`, REST GUI on
  `127.0.0.1:8385` (API key in `config.xml` there) — deliberately not the
  user-managed default 8384.
- Transport is pinned to `tcp://<peer-tailscale-magicdns>:22000` with
  global/local discovery, relays, and NAT traversal all disabled — traffic
  cannot leave the tailnet.
- File versioning (staggered, 30-day) is on for every managed folder;
  versions live under `~/.openbase/sync-versions/<folder-id>/`. This is the
  undo net for uncommitted work — check it before declaring data lost.
- Write lease: the device with recent voice/agent activity stays
  send-receive; idle peers flip receive-only. For deliberate simultaneous
  work on both machines set `lease_mode: manual` via `PUT
  /api/sync/settings/`.
- Conflict records (branch divergences and `*.sync-conflict-*` files) are
  at `GET /api/sync/conflicts/`, resolvable via the console/iOS Sync pages
  or the CLI.

## Device Access Setup (walk the user through this)

Sync problems are usually diagnosed from one machine while the other
misbehaves, so establish access first. Prompt the user to:

1. **Tailscale on both devices, same tailnet.** Verify with
   `tailscale status` and that each device resolves the other's MagicDNS
   name (`ping <device>.<tailnet>.ts.net`). Openbase Coder pairing already
   requires this; sync reuses it.
2. **Public-key SSH from the laptop to the remote machine** (bidirectional
   if the user works from both). On the remote Mac enable Remote Login
   (System Settings → General → Sharing, or
   `sudo systemsetup -setremotelogin on`), then from the laptop:

   ```bash
   ssh-keygen -t ed25519    # if no key yet; accept defaults
   ssh-copy-id <user>@<device>.<tailnet>.ts.net
   ssh -o BatchMode=yes <user>@<device>.<tailnet>.ts.net true  # must succeed silently
   ```

   Use the Tailscale MagicDNS hostname, not a LAN IP — it works from
   anywhere on the tailnet. Note that over non-interactive SSH the remote
   keychain stays locked: HTTPS git fetches of private repos will fail
   there even when they work in a local session on that machine.
3. **Keep sync traffic on the tailnet, not the public internet.** For the
   managed instance this is guaranteed by construction. For a user-managed
   Syncthing, check each device's config: peer device addresses should be
   pinned `tcp://<magicdns-or-tailscale-ip>:22000` (not `dynamic`), and
   `globalAnnounceEnabled`, `relaysEnabled`, `natEnabled` should be
   `false`. A relay in the connection list (`/rest/system/connections`
   showing a `relay://` address) means data is transiting third parties.
4. **Same GUI credentials on both devices** (user-managed instances only —
   the managed instance is localhost + API key and needs no password).
   Set the same GUI username/password on each device's Syncthing web UI
   (Actions → Settings → GUI) so the user — and agents walking them
   through fixes — can administer either side without hunting for
   per-device passwords. REST calls authenticate with the `<apikey>` from
   that device's `config.xml` regardless.

## Diagnosing Sync Problems

REST quick kit (add `-H "X-API-Key: <key>"`; key is in the instance's
`config.xml`; port 8385 managed, 8384 user-managed default):

- Folder health: `GET /rest/db/status?folder=<id>` — `state` should reach
  `idle` with `needTotalItems: 0`.
- What is stuck: `GET /rest/db/need?folder=<id>&page=1&perpage=50`.
- Force rescan / reload ignores: `POST /rest/db/scan?folder=<id>`.
- Service-level errors: `GET /rest/system/error`.

Common failures and their actual causes:

- **`*.sync-conflict-*` files** — both machines edited the file in the same
  window. Diff the conflict copy against the kept file, merge what matters,
  delete the copy. Code sync records these at `/api/sync/conflicts/`;
  sweep for strays with `find <folder> -name "*.sync-conflict-*"`.
- **"directory has been deleted on a remote device but contains changed
  items"** or a symlink/dir replacement that never completes — the
  directory contains locally *ignored* files (venv, node_modules) and the
  ignore patterns lack the `(?d)` prefix, so Syncthing may not delete them
  to complete the operation. Prefix junk patterns with `(?d)`; if the
  leftover content is regenerable (a stray `.venv`), delete it and rescan.
- **Ignore added but files still listed/synced** — ignores only apply to
  that device, and only after a rescan; already-synced copies on the peer
  are not deleted by ignoring. Apply the pattern on both sides, rescan,
  and clean up existing copies manually.
- **"folder marker missing"** — the folder's `.stfolder` marker directory
  was deleted (cleanup tools do this). If the folder contents are intact,
  recreate it (`mkdir <folder>/.stfolder`) and rescan; the scary
  data-loss wording usually just means the marker is gone.
- **"folder path missing"** — the configured directory no longer exists on
  this device. Recreate it to resume, or pause/remove the folder on both
  devices if it is genuinely retired.
- **Peer completion stuck below 100%** — check the *peer's* own
  `/rest/db/status`: it is usually still scanning (initial scans of large
  trees take many minutes) or working through a large `needBytes` backlog.
  Slow-but-moving is healthy; investigate only if `needTotalItems` is
  frozen across several minutes, then read the top of its need list.
- **Peer shows connected on one side only, or dials fail repeatedly with
  "Failed to exchange Hello messages … EOF"** — usually a half-open zombie
  connection: one side still believes an earlier connection (to a process
  that has since restarted) is alive and silently drops new dials as
  duplicates. Restart the Syncthing instance on the side that logs nothing
  during the other side's dial attempts.
- **A repo shows the same changes as dirty on both machines** — correct
  behavior, not a bug: working files synced while each machine keeps its
  own `.git`. Commits made on one machine propagate via the reconciler
  (or a plain `git fetch && git reset` when trees already match). Never
  "fix" this by syncing `.git`.
- **Stale months-old file content reappearing** — a working-tree echo from
  a peer that was offline with old state. Check file versioning
  (`~/.openbase/sync-versions/` or `.stversions/`) to restore, and check
  whether the stale peer skipped its rescan.

## Retrofitting A User-Managed Syncthing That Syncs Code

If the user already syncs code folders with their own Syncthing:

1. Add the VCS block to the ignore file **on every device**, at the top,
   before broader patterns (first match wins):

   ```text
   (?d).git
   (?d)**/.git
   (?d).jj
   (?d).hg
   ```

   Bare names match at every level including the folder root; `**/name`
   alone misses the root.
2. Rescan and verify via `GET /rest/db/ignores` on **each** device (rule 2
   above: the ignore file itself does not sync).
3. Prefix regenerable junk (`node_modules`, `venv`, build outputs,
   `__pycache__`, `.DS_Store`) with `(?d)` so deletions propagate.
4. Enable staggered versioning on code folders as the undo net for
   uncommitted work.
5. Recommend migrating to `openbase-coder sync`, which owns all of the
   above and cannot regress silently (doctor checks the managed ignores).

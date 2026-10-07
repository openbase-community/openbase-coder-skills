---
name: openbase-file-sync
description: >-
  Use this skill when setting up or diagnosing file sync between a user's computers with Openbase Sync: pairing a laptop (edge) with an always-on Mac mini, desktop or DevSpace (hub), choosing which folders sync, sync conflicts, migrating from the previous Openbase code sync, SSH access between devices, or why git repos must never sync their .git directories.
version: 0.2.0
---

# Openbase File Sync

Keep a user's second machine a live replica of their working files — dirty changes just appear on the other device, with no commits required — without ever letting file sync corrupt git state.

## When To Use

Use this skill when:

- the user wants files synced between two of their machines (laptop ⇄ desktop/Mac mini, laptop ⇄ Cloud DevSpace)
- setting up or checking Openbase Sync (`openbase-coder sync-daemon`, `openbase-coder sync`)
- diagnosing sync conflicts, a disconnected peer, or changes that do not arrive on the other machine
- moving a machine off the previous Openbase code sync
- the user asks about SSH access between their devices for sync or administration

## Hard Rules

1. **`.git` (and any VCS metadata: `.jj`, `.hg`) must NEVER travel as plain files.** Copying a live `.git` file by file tears refs and index state and silently corrupts commits. Openbase Sync never file-syncs `.git`; it moves commits, branches and worktrees as git. Never work around this.
2. **Conflicts are records, not overwrites.** When both machines changed the same thing incompatibly, Openbase Sync keeps both and records a conflict. Resolve it deliberately (keep mine / take theirs); never "fix" it by copying files over by hand.
3. Secrets (`.env`, keys) inside synced folders sync **on purpose** — that is a feature git cannot provide. Do not "fix" it.
4. Delete state by moving it to trash, never with `rm -rf` on both machines; a deletion on one machine propagates to the other.

## Route To The Product Docs

Don't restate the product docs here:

- **How it works** — hub/edge, roots, placement and pins, git as git, conflicts, thread and skills sync, migration — is owned by *Sync Between Your Computers* (`cli/docs/code-sync.md`, published at docs.openbase.cloud).
- **Commands** — `openbase-coder sync status|conflicts|resolve|migrate-from-syncthing` (`cli/docs/commands/sync.md`) and `openbase-coder sync-daemon configure|install-binary|status|conflicts|resolve|disable` (`cli/docs/commands/sync-daemon.md`).

## Setup Walkthrough

1. **Both devices on Openbase VPN.** Each must reach the other's VPN address. The hub is the machine that stays on.
2. **Install the sync binaries** Openbase provides on both machines with `openbase-coder sync-daemon install-binary ...`.
3. **Configure the hub** with its VPN address and the folders to sync; keep the printed pair secret:

   ```bash
   openbase-coder sync-daemon configure --role hub --listen <hub-vpn-ip> \
     --root ~/Projects --with-product-folders
   ```

4. **Configure the edge** with the same roots:

   ```bash
   openbase-coder sync-daemon configure --role edge --peer <hub-vpn-ip> \
     --pair-secret <secret> --root ~/Projects --with-product-folders
   ```

   `--with-product-folders` keeps Openbase thread sync and skills sync working. Use the same home-relative layout on both machines.
5. **Verify** with `openbase-coder sync status` on each machine (a peer is listed) or the console Sync page.

## Migrating From The Previous Code Sync

Machines that used the earlier Openbase code sync (a `code-sync` service, `~/.openbase/code-sync`) need a one-time migration on **each** machine:

```bash
openbase-coder sync migrate-from-syncthing          # review the plan
openbase-coder sync migrate-from-syncthing --apply
```

It removes the old service, moves its state into `~/.openbase/trash/`, and adds the previously synced folders as Openbase Sync roots (or prints the `sync-daemon configure` command if Openbase Sync is not set up yet). Old custom ignore rules are not carried over; the command prints how many were left behind.

Only after `--apply` has run on **every** machine, remove the old marker and ignore files with `openbase-coder sync migrate-from-syncthing --apply --remove-markers`. Doing it earlier is dangerous: Openbase Sync would carry the deletion of an ignore file to a machine whose old sync is still running, and that engine would then start syncing `.git`. Ask before running `--apply` on a user's machine.

## Diagnosing Sync Problems

- **Daemon not answering** — `openbase-coder services start sync-daemon`, then `openbase-coder services logs sync-daemon`.
- **No peer** — the hub is off, asleep, or unreachable over Openbase VPN. Check both machines' VPN status; the edge reconnects by itself.
- **A change did not arrive** — check `openbase-coder sync conflicts` first: a conflicted path waits for a decision. Confirm the folder is inside a configured root (`openbase-coder sync status`).
- **A branch looks different on each machine** — a diverged branch shows up as a conflict; pick a side with `openbase-coder sync resolve <id> --keep-local|--use-remote`.
- **A commit or build fails mysteriously** — check conflicts and compare `HEAD` with `origin/<branch>` before committing.

## Device Access Setup

Sync problems are usually diagnosed from one machine while the other misbehaves, so establish access first:

1. **Public-key SSH from the laptop to the remote machine** (bidirectional if the user works from both). On the remote Mac enable Remote Login (System Settings → General → Sharing, or `sudo systemsetup -setremotelogin on`), then from the laptop:

   ```bash
   ssh-keygen -t ed25519    # if no key yet; accept defaults
   ssh-copy-id <user>@<remote-vpn-hostname>
   ssh -o BatchMode=yes <user>@<remote-vpn-hostname> true  # must succeed silently
   ```

   Use the VPN hostname, not a LAN IP, so it works from anywhere. Over non-interactive SSH the remote keychain stays locked: HTTPS git fetches of private repos fail there even when they work locally on that machine.

## User-Managed File Sync Tools

If the user syncs code folders with a file-sync tool of their own, make sure it excludes `.git`, `.jj` and `.hg` on **every** device (ignore files usually do not sync themselves), verify the running instance actually applies the patterns, and recommend moving those folders to Openbase Sync. Never let the same folder be synced by both tools at once.

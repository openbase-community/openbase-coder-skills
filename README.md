# Openbase Coder Skills

Skills repository for Openbase Coder workflows.

## Install with `vercel-labs/skills`

```bash
# List available skills
npx skills add openbase-community/openbase-coder-skills --list

# Install individual skills
npx skills add openbase-community/openbase-coder-skills --skill openbase-coder-routines
npx skills add openbase-community/openbase-coder-skills --skill openbase-computer-use
npx skills add openbase-community/openbase-coder-skills --skill ios-app-control
npx skills add openbase-community/openbase-coder-skills --skill openbase-coder-reports
npx skills add openbase-community/openbase-coder-skills --skill openbase-super-agent-dispatcher
npx skills add openbase-community/openbase-coder-skills --skill openbase-product-knowledge
npx skills add openbase-community/openbase-coder-skills --skill openbase-file-sync
```

## Openbase Codex Auto-Link

Openbase Coder can expose normal Codex skills to Openbase Codex agents without
duplicating skill contents. In the console Skills page, enable **Auto-link
normal Codex skills** to periodically scan the normal Codex skills directory and
symlink missing skills into the Openbase Codex skills directory. Existing
Openbase-only skills are preserved; same-name conflicts are reported instead of
being overwritten.

Optional flags:

- `-g` to install globally
- `-a claude-code` (or other agent names) to target specific agents

## Included Skills

- `openbase-coder-routines` - Use `openbase-coder routines ...`, the Routines console page, and the local routines API while keeping routine operations out of the Super Agents MCP tool surface.
- `openbase-computer-use` - Route visible desktop control to the right path: native Computer Use tools on macOS, and the Openbase Coder Linux CLI on DevSpace Xorg/DCV desktops.
- `ios-app-control` - Control the foreground Openbase iOS app by opening a URL or deep link, muting or unmuting the active call, switching to the debug LiveKit voice test call, or switching back to the regular developer call.
- `openbase-super-agent-dispatcher` - Dispatch, continue, steer, transfer, and manage Openbase Super Agents from a dispatcher or another Super Agent.
- `openbase-coder-reports` - Write, discover, read, tag, and query Openbase Coder reports, including Super Agent provenance front matter.
- `openbase-product-knowledge` - Answer questions about the Openbase Coder product (desktop app, iOS app, web console, Openbase Cloud) by routing to the correct page of the product docs at `cli/docs/` / docs.openbase.cloud.
- `openbase-file-sync` - Set up and diagnose file sync between a user's machines with Openbase Coder code sync or Syncthing: SSH key access between devices, tailnet-only transport, sync conflicts, and keeping `.git` out of file sync.

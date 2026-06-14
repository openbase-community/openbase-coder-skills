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
```

Optional flags:

- `-g` to install globally
- `-a claude-code` (or other agent names) to target specific agents

## Included Skills

- `openbase-coder-routines` - Use `openbase-coder routines ...`, the Routines console page, and the local routines API while keeping routine operations out of the Super Agents MCP tool surface.
- `openbase-computer-use` - Route visible desktop control to the right path: native Computer Use tools on macOS, and the Openbase Coder Linux CLI on DevSpace Xorg/DCV desktops.
- `ios-app-control` - Control the foreground Openbase iOS app by opening a URL or deep link, muting or unmuting the active call, switching to the debug LiveKit voice test call, or switching back to the regular developer call.

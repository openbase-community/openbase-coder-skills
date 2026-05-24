# Openbase Coder Skills

Skills repository for Openbase Coder workflows.

## Install with `vercel-labs/skills`

```bash
# List available skills
npx skills add openbase-community/openbase-coder-skills --list

# Install individual skills
npx skills add openbase-community/openbase-coder-skills --skill openbase-coder-routines
npx skills add openbase-community/openbase-coder-skills --skill openbase-computer-use-cli
```

Optional flags:

- `-g` to install globally
- `-a claude-code` (or other agent names) to target specific agents

## Included Skills

- `openbase-coder-routines` - Use `openbase-coder routines ...`, the Routines console page, and the local routines API while keeping routine operations out of the Super Agents MCP tool surface.
- `openbase-computer-use-cli` - Use the Openbase Coder computer-use CLI to control the user's Mac through the LiveKit screen-share companion.

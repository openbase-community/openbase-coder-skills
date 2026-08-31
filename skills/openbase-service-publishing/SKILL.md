---
name: openbase-service-publishing
description: Use this skill when a user or agent needs to open, share, preview, or reach a local development website or single-port HTTP service from another device over Openbase VPN, or when advice would otherwise tell a phone to use localhost.
---

# Openbase Service Publishing

Use Openbase's built-in private publication command for a local HTTP service:

```bash
openbase-coder service publish <memorable-name> <local-port>
```

Do not tell a different device to open `localhost`; that name points back to
the device doing the browsing. Return the exact tailnet URL printed by the
command. The command uses Tailscale Serve privately and never enables Funnel.

Before publishing:

1. Ensure the app listens on `127.0.0.1` at one TCP port.
2. Pick a short lowercase name containing letters, numbers, or hyphens.
3. Run `publish`. Do not invent a `.local` name; `.local` is reserved for mDNS.

Persistence is a separate user choice. When the CLI asks whether to restore
the gateway at login with launchd, relay that question and wait for the user's
answer. Never pass `--persist` on the user's behalf without explicit approval.
For non-interactive work, omit the flag and explain that publication lasts for
the current login session. Tailnet-facing persistent ports must remain in the
uncommon dynamic/private range selected by the command.

Useful commands:

```bash
openbase-coder service list
openbase-coder service unpublish <name>
```

This feature requires **Openbase VPN**. **Openbase Direct** intentionally
carries only Openbase app traffic and cannot make arbitrary sites available to
a phone browser.

Treat Docker multi-port projects as the exception. Publish a single web ingress
when one exists; otherwise use the project's tailnet/container networking and
document each required port. Do not imply that one HTTP publication carries
database, UDP, or other independent ports.

Dedicated per-service DNS names remain a provider-admin boundary. Openbase's
local command uses the computer's existing MagicDNS identity and a memorable
URL path. Do not claim to have created Headscale extra DNS records or hosted
Tailscale Services.

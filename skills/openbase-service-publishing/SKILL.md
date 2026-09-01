---
name: openbase-service-publishing
description: Use this skill when a user or agent needs to open, share, preview, or reach a local development website or single-port HTTP service from another device over Openbase VPN, or when advice would otherwise tell a phone to use localhost.
---

# Openbase Service Publishing

Use Openbase's built-in private publication command for a local HTTP service.
Prefer an explicit dedicated hostname when Openbase VPN advertises support:

```bash
openbase-coder service publish <memorable-name> <local-port> --mode hostname
```

Do not tell a different device to open `localhost`; that name points back to
the device doing the browsing. Return the exact tailnet URL printed by the
command. A hostname publication looks like
`http://<service>.<node>.netmesh.openbase.cloud/`. It forwards the incoming
path and query unchanged to the service's root and never adds, strips, or
rewrites a `/services/...` prefix. The command uses private Serve routing and
never enables Funnel.

Hostname mode is opt-in while it rolls out. It must fail closed unless both the
Openbase Cloud DNS allocator and the signed local helper advertise the exact
capability, and unless the allocated hostname resolves to this node. Do not
invent a hostname or use a partially configured result. The compatibility
fallback remains the existing dynamic/private-port command:

```bash
openbase-coder service publish <memorable-name> <local-port>
```

That default preserves its existing URL and path behavior for current users.
Use `--mode auto` only when the user explicitly accepts a hostname attempt with
a safe fallback to the existing dynamic mode.

Before publishing:

1. Ensure the app listens on `127.0.0.1` at one TCP port.
2. Pick a short lowercase name containing letters, numbers, or hyphens.
3. Run `publish`. Do not invent a `.local` name; `.local` is reserved for mDNS.

Persistence is a separate user choice. When the CLI asks whether to restore
the gateway at login with launchd, relay that question and wait for the user's
answer. Never pass `--persist` on the user's behalf without explicit approval.
For non-interactive work, omit the flag and explain that publication lasts for
the current login session. A separately configured local app may also need its
own launchd job; ask before making the app always-on as well as before making
the publication always-on. Dynamic tailnet-facing ports must remain in the
uncommon private range selected by the command. Hostname publication uses the
signed helper's private HTTP route and keeps its local proxy on an uncommon
loopback port. Neither mode may bind the app or proxy to `0.0.0.0`.

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

Dedicated per-service DNS names are an Openbase VPN control-plane capability,
not a local naming trick. The authenticated owner-scoped Cloud API allocates
the Headscale record; the signed helper installs only the matching hostname
route. Official Tailscale, unknown providers, and Openbase Direct must reject
hostname publication before changing local state.

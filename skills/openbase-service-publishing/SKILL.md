---
name: openbase-service-publishing
description: Use this skill when a user or agent needs to open, share, preview, or reach a local development website or single-port HTTP service from another device over Openbase VPN, or when advice would otherwise tell a phone to use localhost.
---

# Openbase Service Publishing

Use Openbase's built-in private publication command for a local HTTP service. A dedicated private hostname is the default:

```bash
openbase-coder service publish <memorable-name> <local-port>
```

Do not tell a different device to open `localhost`; that name points back to
the device doing the browsing. Return the exact tailnet URL printed by the
command. A hostname publication looks like
`https://<service>.<account-namespace>.vpn.obs.so/`. It forwards the incoming
path and query unchanged to the service's root and never adds, strips, or
rewrites a `/services/...` prefix. The command uses private Serve routing and
never enables Funnel.

Publication must fail closed unless Cloud advertises account-private DNS, the signed helper advertises the current account-namespace routing contract, and the allocated name resolves exclusively to this node. Never publish service names through Headscale's shared extra-record list. The independent VPN-only DNS resolver must identify the querying peer and answer only its account's records.

There is one supported publication mode. Do not use the retired `--mode`, `--tailnet-port`, dynamic-port fallback, or prefix-based modes. A missing capability requires updating/configuring the components, not inventing a URL or a workaround. Old registry entries are for cleanup only; remove them before upgrading the helper and republish on their root hostname.

Namespaces belong to accounts, not devices. Service names are unique within an account. To move a service, unpublish it from its current device first; publishing the same name on another owned device retains its URL. The opaque namespace is a label, never an authorization credential.

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
the publication always-on. Hostname publication uses the
signed helper's VPN-only TCP 443 route to a device-local TLS ingress. Port 80 redirects to HTTPS. Neither the app nor proxy may bind to `0.0.0.0`.

The device obtains and automatically renews a Let's Encrypt DNS-01 wildcard certificate for `*.<account-namespace>.vpn.obs.so`. Only the random account namespace appears in Certificate Transparency, not service names. Keys stay in owner-only files on the serving device; Cloudflare credentials stay on Cloud, whose authenticated broker permits validation only for the caller's account. TLS does not make a service publicly reachable or replace VPN account isolation. Do not bypass certificate validation. Renewal runs while a publication is active and at startup, without installing an extra persistent job. Apps moving from HTTP need their trusted origins updated to HTTPS.

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
the owner-scoped private DNS record; the signed helper installs only the matching hostname
route. Official Tailscale, unknown providers, and Openbase Direct must reject publication before changing local state.

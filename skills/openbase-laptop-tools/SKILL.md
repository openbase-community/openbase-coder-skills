---
name: openbase-laptop-tools
description: >-
  Use this skill when you run on a hub machine (an always-on Mac mini,
  desktop, or Cloud DevSpace) and a task might need the user's laptop: its
  screen, clipboard, speakers, browser, iOS simulator or attached devices, a
  local port such as Chrome's DevTools port, or offered MCP servers named
  `*-laptop` (for example `computer-laptop`). Covers `edge status`, `edge
  run`, `edge where`, `edge forward`, and the MCP gateway. These are options
  you choose; nothing is routed to the laptop automatically.
version: 0.1.0
---

# Openbase Laptop Tools

When the user's code and agents run on a hub, the user is usually somewhere
else, in front of their laptop. Builds, tests, and git belong on the hub. A
few things only make sense on the laptop: opening a page the user should see,
putting text on their clipboard, a notification, the iOS simulator or a
plugged-in phone, their logged-in browser, their screen.

Openbase gives you ways to reach the laptop **when you decide to**:

- the `edge` command, for a small allowlist of display-bound commands, file
  presence checks, and port forwards;
- MCP servers the user offered from the laptop, listed in your tools with a
  `-laptop` suffix.

## You are in charge

- **Choose per action.** Decide for each step whether it belongs on the hub
  or the laptop. Default to the hub; reach for the laptop when the user must
  see, hear, or touch the result, or when the resource only exists there.
- **Never wrap commands automatically.** Do not create aliases, shims, PATH
  entries, hooks, or scripts that send commands to the laptop on their own.
  Every laptop action is an explicit `edge` call or an explicit call to a
  `*-laptop` tool.
- **Say where it ran.** Tell the user what happened on the laptop ("opened
  the preview on your laptop", "copied to your laptop's clipboard"), and
  what ran on the hub.
- **Do not silently fall back.** If the laptop is unreachable, say so. Do not
  quietly run the same command on the hub instead: `open` on a headless hub
  shows nothing to anyone.
- **Respect the laptop.** Only do what the task needs. Do not change its
  volume, settings, or running apps beyond the request, and ask before
  restarting the user's browser or anything else they have open.

## Check first: `edge status`

```bash
edge status
```

Prints JSON. `self.role` is `hub` or `edge` (an `edge` is the display
machine, so if that is you, you are already on the laptop and can run things
locally). `peers` lists connected machines with `has_display` and
`exec_allow`, and `display_peer` names the laptop when one is online.

If `edge` is not found, try `~/.openbase/bin/edge`. If it is missing there
too, or it reports that the sync daemon is not running, the user has not set
up hub/edge sync on this machine: tell them, and do not install or configure
it yourself.

## Run a display-bound command: `edge run`

```bash
edge run open https://github.com/org/repo/pull/42
edge run open ./build/report.html
echo "deploy key rotated" | edge run pbcopy
edge run pbpaste
edge run say "Tests passed"
edge run osascript -e 'display notification "Build finished" with title "Openbase"'
edge run xcrun simctl openurl booted myapp://settings
edge run devicectl list devices
```

- Only allowlisted commands run: `open`, `pbcopy`, `pbpaste`, `say`,
  `osascript` (only `display notification`, `display dialog`, or
  `display alert`), `devicectl`, and `xcrun simctl`. `edge status` shows the
  laptop's actual list. Anything else is refused; do not try to smuggle
  commands through `open` or `osascript`.
- A URL opens in the laptop's browser, so `localhost` there is the laptop,
  not the hub. To show the user a dev server running on the hub, publish it
  first (the `openbase-service-publishing` skill) and open the URL it prints.
- Path arguments are made absolute and refer to the same path on the laptop.
  That only works for synced files that have arrived there; check with
  `edge where` first when it matters.
- Piped stdin is forwarded to the command; stdout, stderr, and the exit code
  come back.
- **Exit code 75 means the laptop is offline** (asleep, closed, or
  disconnected). It fails fast on purpose. Tell the user; do not retry in a
  loop. Other non-zero codes are the command's own, or `1` when the request
  was refused or the daemon is not running here.
- It is for quick, visible actions. Do not run long jobs on the laptop.

## Is this file on the laptop: `edge where`

```bash
edge where ./build/report.html
```

Prints JSON: `synced` (under a synced folder), `known`, `placement`, and
`agreed` (both machines hold the same content). If it is not synced or not
yet agreed, the laptop may not have your latest version: wait, or tell the
user, rather than opening a stale file.

## Ports: `edge forward`

```bash
edge forward start 9222          # laptop 127.0.0.1:9222 -> hub 127.0.0.1:9222
edge forward start 9222 --local 19222
edge forward list
edge forward stop 9222
```

A forward makes a port that listens on the laptop's loopback reachable on the
hub's loopback until you stop it. The laptop allows only the ports it lists
(by default 9222, Chrome's remote debugging port). Exit code 75 again means
the laptop is offline.

Driving the user's own logged-in Chrome from the hub with Playwright:

1. Chrome must already be listening on the laptop
   (`--remote-debugging-port=9222`). If it is not, ask the user to start it
   that way. Do not quit or relaunch their browser yourself.
2. `edge forward start 9222`
3. Connect: `chromium.connectOverCDP("http://127.0.0.1:9222")` (Python:
   `p.chromium.connect_over_cdp("http://127.0.0.1:9222")`).
4. Work in a new tab or context you created, leave the user's tabs alone,
   and `edge forward stop 9222` when done.

## Offered MCP servers: `*-laptop`

The user can offer MCP servers that run on their laptop to agents on the hub.
They appear in your tool list with a `-laptop` suffix, for example
`computer-laptop`: computer control of the **laptop's** screen. A tool with
the same name and no suffix acts on the hub's own screen, which is usually
not what the user is looking at.

- Calling one opens a session on the laptop for as long as your session uses
  it. Use it like any other MCP server, and only when the task calls for it.
- If it is unavailable or fails to start, the laptop is offline, does not
  serve that server, or is signed in to a different account. Tell the user.
- Offering and serving change the user's configuration. Suggest the commands
  and run them only when the user asks:
  - on the laptop: `openbase-coder mcp-gateway serve add computer`
  - on the hub: `openbase-coder mcp-gateway offer computer --peer <laptop name>`
    (`openbase-coder mcp-gateway offered` lists what is offered). New sessions
    pick it up.

The product docs page "Laptop Tools for Remote Agents" (`laptop-tools.md` in
the Openbase docs) describes the same features for users.

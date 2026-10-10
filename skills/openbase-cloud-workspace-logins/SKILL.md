---
name: openbase-cloud-workspace-logins
description: >-
  Use this skill when an agent must sign a command-line tool in (gh, Codex, Claude Code, gcloud, Heroku, an MCP server's OAuth, or any CLI that "opens a browser" to log in) from an Openbase cloud workspace, or from any host whose browser is on another device such as the user's phone. Covers device-code and paste-code flows, `openbase-coder browser open`, and pasting back a failed `http://localhost:<port>/...` callback address.
version: 0.4.0
---

# Logging CLIs In from a Cloud Workspace

In an Openbase cloud workspace there is no browser next to you: the user signs in on their phone or another computer. A login whose last step redirects to `http://localhost:<port>/...` therefore ends on the wrong device, because `localhost` on the phone is the phone. Prefer a device-code flow when the CLI offers one. For a loopback redirect, request localhost forwarding and check the phone’s acknowledgement; use paste-back when forwarding is unavailable. A successful device-code login does not test localhost forwarding.

Login commands wait for the user. Run them so they keep running while you talk to the user (a background job or a separate terminal session), relay the URL and any code to the user exactly as printed, and tell them what to do on their side. Never ask the user for a password, and never type one on their behalf.

Check the tool's existing authentication first, as the workspace user. If the intended account already works, continue the user's task without starting another login. Do not create a second pending login while an earlier one is still valid.

## Install missing CLIs and preserve existing authentication

Run `command -v gh`, `command -v heroku`, and `command -v gcloud` as the workspace user. Install only the tool the task needs. After installation run its `--version` and login `--help`; do not assume a missing executable means the account is unauthenticated. Keep existing credential directories and environment overrides intact.

Before downloading a large SDK, check free space on the installation, extraction, temporary-download and package-cache filesystems with `df -h`; allow room for both the archive and expanded installation. A cloud workspace can have a nearly full root disk while its persistent data volume has ample space. If `/data` is available, verify that it is persistent and writable, then choose a dedicated tools directory there when root space is insufficient. Put the download and extraction staging area on that volume too, and add the installed tool's `bin` directory to the current `PATH`. Do not move credentials or overwrite an existing installation. If an install fails for lack of space, remove only the partial download/extraction created by that attempt, subject to the user's cleanup rules, and retry in the checked destination; never clear unrelated caches or files to make room.

| Tool | Install if missing | Check before login | Login command |
|---|---|---|---|
| GitHub | On macOS: `brew install gh`. On Debian/Ubuntu with a package available: `sudo apt-get update && sudo apt-get install -y gh`; otherwise use the [official Linux repository instructions](https://github.com/cli/cli/blob/trunk/docs/install_linux.md). | `gh auth status --hostname github.com`, then `gh api user --jq .login` for the intended account. | `gh auth login --web --hostname github.com --git-protocol https` |
| Heroku | On macOS: `brew tap heroku/brew && brew install heroku`. With Node/npm available: `npm install --global --prefix "$HOME/.local" heroku`, then add `$HOME/.local/bin` to the current `PATH`. Other platforms: [Heroku CLI installation](https://devcenter.heroku.com/articles/heroku-cli). | `heroku auth:whoami`; check whether `HEROKU_API_KEY` is set without printing it. | `heroku login`; if needed, `heroku login --browser openbase-browser` captures the ordinary remote login URL. |
| Google Cloud | On macOS: `brew install --cask gcloud-cli`. On Linux follow the [official architecture-specific archive or package instructions](https://docs.cloud.google.com/sdk/docs/install-sdk); run the archive's `install.sh --quiet`, add its `bin` directory to `PATH`, and skip `gcloud init` until authentication is checked. | `gcloud auth list --filter=status:ACTIVE --format='value(account)'`; check the configured account and credential overrides without printing token values. | Prefer `gcloud auth login --no-launch-browser` on headless Linux for the phone verification-code flow. |

Google Cloud CLI selects its browser flow using its own environment checks. On headless Linux, `BROWSER=openbase-browser` alone does not force a localhost callback, even with `--launch-browser`; it can still choose a remote verification-code flow. Follow the actual printed instructions. Before treating a gcloud login as a loopback-relay test, confirm both the localhost redirect in its authorization URL and the real CLI-owned listener. A printed provider URL alone is not evidence of callback forwarding or completed authentication.

Do not clear credential files, run logout/revoke, change an already working account, or copy another machine's credentials to make a test pass. Google application-default credentials are separate from the gcloud CLI account: only use `gcloud auth application-default login` when the user's task needs ADC. A status command failing due to network or an unavailable credential helper is not proof that a new login is required; resolve that condition first.

## User-action handoff

Start an interactive login with immediate background execution or a short initial tool yield; do not wait for the foreground tool's multi-minute timeout before reading its output. Read the output as soon as it appears. A background task ID is not the device code. Never block on `TaskOutput` or repeatedly poll while the user has not yet received the code.

Put the exact current device code, login URL, and next action together in the final user-visible reply, even if you already sent them as intermediate commentary or opened the browser. During an active voice session, also promptly say the code and next action using `openbase-coder user say "<your agent name>" "<short instruction with the device code>"`; opening the page does not speak the code. Use your actual speaking name and read the device-code characters distinctly. Never speak passwords, tokens, or localhost callback URLs containing secrets.

Say plainly that you are waiting for the user's authorization. A login shell can remain alive after the model's reply finishes; that is a background task, not evidence that productive coding continues. End your response after delivering the instructions; do not keep doing blocking tool waits just to hold the conversation open. Preserve the login process until success, expiry, or an explicit cancellation. When the user returns, check authentication again and continue the already requested task. If the code expired, start one replacement flow and relay its new code in the same way.

## 1. Prefer device-code and paste-code flows

These finish on the provider's servers or by pasting a code into the terminal, so the browser can be on any device:

| Tool | Command | What the user does |
|---|---|---|
| GitHub CLI | `gh auth login --web --hostname github.com` (device code is the default web flow) | Opens `https://github.com/login/device` and enters the one-time code it prints. |
| Codex | `codex login --device-auth` | Opens the printed URL and enters the code it shows. |
| Claude Code | `claude auth login`, or `/login` inside `claude` | Opens the printed URL, signs in, and sends you the code the page shows; paste it into the waiting prompt. |
| Google Cloud | `gcloud auth login --no-launch-browser` (also `gcloud auth application-default login --no-launch-browser`) | Opens the printed URL, signs in, and sends you the verification code to paste into the prompt. |
| Google Cloud, remote bootstrap | `gcloud auth login --no-browser` | Runs the printed command on a computer that has gcloud and a browser, then sends you the output to paste into the prompt. |
| Heroku and other browser-link logins | `heroku login` | Opens the printed link on any device; the CLI polls the provider and finishes by itself. |

If a tool has a `--device-auth`, `--device-code`, `--no-browser`, `--no-launch-browser`, or "paste the code" option, use it. Check `<tool> login --help` before assuming it does not.

### GitHub CLI: sign in on the workspace, approve on the phone

Run these commands as the same workspace user that runs the coding agent, in the same environment. A management shell may run as root; signing root in does not sign the workspace user in. Check `command -v gh` and `gh auth status --hostname github.com` first. If an account is already authenticated, verify it is the requested account before replacing or switching it. Environment-provided `GH_TOKEN` or `GITHUB_TOKEN` can override a saved login; check whether they are set without printing their values, and do not copy another computer’s token into the workspace.

```bash
gh auth login --web --hostname github.com --git-protocol https
```

Keep that process alive, relay its one-time device code, and open `https://github.com/login/device` on the phone. An interactive command may prompt to launch the browser; a command without a terminal may only print the URL. If the page does not open automatically, use `openbase-coder browser open https://github.com/login/device`. The phone approves GitHub CLI for the intended account. If authorized to control the phone, an agent can enter the device code and complete the approval; the user handles any password, passkey, or two-factor challenge directly.

After the command finishes, verify from the same workspace user:

```bash
gh auth status --hostname github.com
gh api user --jq .login
```

Use the returned login to confirm the account. Do not run `gh auth token` or print the credential file. GitHub CLI uses a device-code flow: it polls GitHub and does not redirect to localhost, so do not add `--callback-port` for this login. Workspace images that bundle GitHub CLI persist its default `~/.config/gh` directory on the workspace data volume. On older images or with an overridden `GH_CONFIG_DIR`/`XDG_CONFIG_HOME`, verify the actual config directory is durable before relying on it surviving an image replacement.

## 2. Send the login page to the phone

```bash
openbase-coder browser open "<login-url>"
```

The command prints the URL, then asks the Openbase app on the user's phone to open it. It prints `Sent to the Openbase app on your phone.` when the app opens the URL or posts a notification to tap; older apps only acknowledge receipt. When a new app reports that neither succeeded, or no app responds, it falls back to Openbase Cloud and prints `Sent a notification to your phone; tap it to open the page.` after Cloud accepts the request. Otherwise it prints a hint to open the URL on any device. Callback setup and socket delivery each wait at most six seconds, and push fallback waits at most eighteen seconds (thirty seconds total, excluding process startup). Delivery failures exit successfully, so the calling CLI can continue. Use `--no-push` to skip Cloud fallback. Malformed URLs, URLs without a scheme and `data:`, `file:`, and `javascript:` URLs are rejected. `openbase-coder user ios open-url <url>` is an alias with callback forwarding disabled.

When the login URL itself is loopback, or its `redirect_uri`/`redirect_url` names `http://localhost:<port>/...`, `browser open` starts a short-lived authenticated relay on embedded-VPN workspaces. The CLI callback must already be listening on loopback and belong to the current workspace user. Openbase records that process and listener, exposes a separate random relay port on the VPN, and gives the phone a per-login capability. The iOS packet-tunnel extension or Android VPN app binds the phone's `127.0.0.1:<port>` and `[::1]:<port>` and forwards the unchanged browser request through the relay. The CLI's own callback port is not exposed directly. Loss of the original listener stops new connections; an accepted callback may finish while the original process is alive and within the ten-minute deadline. Process exit or expiry closes every connection, and delayed notifications cannot restart the original lifetime. Multiple HTTP requests are allowed for logins that begin on a local page.

Only `forward=started` confirms phone-side setup; the new native apps check relay health before acknowledging. `unsupported`, `failed`, `vpn_down`, or no acknowledgement means forwarding is unconfirmed and the paste-back fallback remains available. Use `--callback-port <port>` when the callback port is not in the URL and `--no-forward` to disable setup. Keep the login process alive while the user authorizes. Do not run `service expose <callback-port>` as a substitute: that omits the capability and process binding. Older apps/backends may require upgrading before they can use the authenticated relay.

Openbase cloud workspaces preset `BROWSER` and `GH_BROWSER` to the `openbase-browser` executable, which invokes `openbase-coder browser open`, so CLIs that honour those variables send their login page to the phone without you doing anything. Still relay the printed URL to the user in case the phone could not be reached.

## 3. Universal fallback: paste back the localhost address

For tools that only support a loopback redirect (`http://localhost:<port>/...` or `http://127.0.0.1:<port>/...`), the login page on the phone ends on an address that fails to load. That address carries a one-time code the waiting CLI needs:

1. Start the login and leave it waiting; its local callback listener must stay up.
2. Get the login URL to the user (`openbase-coder browser open "<login-url>"`, or relay it).
3. Ask the user to finish signing in and, when the browser shows a page that cannot be reached at `http://localhost:<port>/...`, copy the full address from the address bar and paste it into this thread.
4. Replay it inside the workspace against the same loopback address, without following redirects:

   ```bash
   curl -sS --max-time 30 "http://localhost:<port>/<path>?code=...&state=..."
   ```

5. Confirm the login finished (the waiting command exits successfully, or the tool's status command shows the account).

Rules for the pasted address:

- Replay it only if its host is `localhost`, `127.0.0.1`, or `[::1]` and its port matches the waiting login. Never fetch a non-loopback address taken from a pasted URL.
- The code in it is single-use and expires within minutes. If the replay fails, start the login again rather than retrying an old address.
- Treat it as a secret. Tell the user to paste it only into this thread and nowhere else, and do not copy it into reports, commits, logs, or other threads.

## Notes

- The port in the redirect belongs to the login process running in the workspace. Do not start your own listener on it.
- If a login needs the user's own computer instead (for example a browser that is already signed in), and that computer is connected as a laptop for this workspace, the `openbase-laptop-tools` skill covers opening pages and forwarding ports there.
- Report plainly whether the login completed. Never say a tool is signed in until its status command or exit code confirms it.

---
name: openbase-cloud-workspace-logins
description: >-
  Use this skill when an agent must sign a command-line tool in (gh, Codex, Claude Code, gcloud, Heroku, an MCP server's OAuth, or any CLI that "opens a browser" to log in) from an Openbase cloud workspace, or from any host whose browser is on another device such as the user's phone. Covers device-code and paste-code flows, `openbase-coder browser open`, and pasting back a failed `http://localhost:<port>/...` callback address.
version: 0.2.0
---

# Logging CLIs In from a Cloud Workspace

In an Openbase cloud workspace there is no browser next to you: the user signs in on their phone or another computer. A login whose last step redirects to `http://localhost:<port>/...` therefore ends on the wrong device, because `localhost` on the phone is the phone. Prefer a device-code flow when the CLI offers one. For a loopback redirect, request localhost forwarding and check the phone’s acknowledgement; use paste-back when forwarding is unavailable. A successful device-code login does not test localhost forwarding.

Login commands wait for the user. Run them so they keep running while you talk to the user (a background job or a separate terminal session), relay the URL and any code to the user exactly as printed, and tell them what to do on their side. Never ask the user for a password, and never type one on their behalf.

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

When the login URL's `redirect_uri` points at `http://localhost:<port>/...` and the workspace runs the embedded tailnet node (every Openbase cloud workspace does), the command also exposes that port on the workspace's VPN address for up to ten minutes (`openbase-coder service expose <port> --one-shot`) and asks the phone to forward its own `localhost:<port>` there. Workspace exposure alone does not confirm a listener on the phone. Only a phone acknowledgement of `started` confirms forwarding was prepared; `unsupported`, `failed`, `vpn_down`, or an absent acknowledgement means use the fallback. Use `--callback-port <port>` when the port is not in the URL and `--no-forward` to skip this. Phone support depends on the installed app version; an app without a callback listener reports `unsupported`. Until the installed app confirms forwarding, the paste-back recipe below still completes the login. See the product documentation's cloud workspace sign-in section for the forwarding contract.

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

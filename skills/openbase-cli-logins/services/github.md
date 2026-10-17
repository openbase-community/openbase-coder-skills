# GitHub CLI (`gh`): fastest phone sign-in

A device-code login: the phone only needs the code and github.com. No localhost forwarding.

## 0. Check first

```bash
gh auth status --hostname github.com
```

If it already shows the account the user wants, stop: they are signed in.

Check for a token in the environment without printing it:

```bash
[ -n "${GH_TOKEN}${GITHUB_TOKEN}" ] && echo "token set in env"
```

If one is set, `gh` uses it and `gh auth login` refuses to store a new login. Tell the user and ask whether to unset it for this session (`unset GH_TOKEN GITHUB_TOKEN`). Never print its value.

## 1. Start it with prompts switched off

```bash
openbase-coder pty start gh-login -- env GH_PROMPT_DISABLED=1 gh auth login --hostname github.com --git-protocol https --web
```

`GH_PROMPT_DISABLED=1` makes gh skip every interactive question (the "Authenticate Git with your GitHub credentials?" one included) and print its code and URL straight away. That matters in a held session: gh's prompt library measures the screen by asking the terminal where the cursor is, and an older gh (2.23, the one in cloud workspaces) waits for that answer forever. `--hostname github.com` and `--git-protocol https` answer the other questions. For SSH use `--git-protocol ssh --skip-ssh-key`.

If the user asks to sign in again although `gh auth status` already shows their account, use the same session with `gh auth refresh --hostname github.com` instead of `gh auth login`; it runs the same device flow and keeps the stored login until the new one completes.

Always through `openbase-coder pty start`, never as a plain shell command: a login run directly by your shell tool is killed when your turn ends, while the user is still on GitHub's page. Start one login and keep it: never start another while a code is still valid (about 15 minutes), because each new run invalidates the previous code.

## 2. Read the code

```bash
openbase-coder pty read gh-login --wait 10
```

The output has `First copy your one-time code: XXXX-XXXX` and `Open this URL to continue in your web browser: https://github.com/login/device`. With prompts off, gh does not open the browser itself and asks for no Enter; it just polls GitHub until the code is entered.

## 3. Show the code on the phone

```bash
printf '%s\n' "XXXX-XXXX" | openbase-coder user phone show-text --text-stdin --label "GitHub code" --open "https://github.com/login/device"
```

Tell the user: "Copy the code, tap Open, paste it and approve." If `show-text` cannot reach the phone, open `https://github.com/login/device` with `openbase-coder browser open` and say the code in your reply (and aloud during a call).

## 4. Wait for it to finish, then verify

```bash
openbase-coder pty read gh-login --wait 30
gh auth status --hostname github.com
```

`Logged in to github.com account <user>` means done. Do not stop the session while the user is still on the page; the code is valid for about 15 minutes.

## Pitfalls

- **Token in the environment:** `GH_TOKEN` or `GITHUB_TOKEN` takes precedence over any stored login (step 0).
- **SSH prompts:** only appear with `--git-protocol ssh`; add `--skip-ssh-key`.
- **Wrong account on the phone:** github.com/login/device approves for whoever is signed in to the phone's browser. Ask the user to check the avatar before approving.
- **Organisation access:** an org with SAML SSO needs an extra "Authorize" click on the approval page.

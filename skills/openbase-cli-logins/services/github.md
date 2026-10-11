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

## 1. Start it with every prompt answered by flags

```bash
openbase-coder pty start gh-login -- gh auth login --hostname github.com --git-protocol https --web
```

- `--hostname github.com` skips the "GitHub.com or Enterprise?" question.
- `--git-protocol https` skips the HTTPS/SSH question and any SSH key upload prompt. If the user wants SSH, use `--git-protocol ssh --skip-ssh-key`.
- `--web` picks the browser/device flow (no token paste).

## 2. Answer the one remaining question

```bash
openbase-coder pty read gh-login --wait 10
```

It asks `Authenticate Git with your GitHub credentials? (Y/n)`. Press Enter (yes):

```bash
printf '\n' | openbase-coder pty send gh-login
```

## 3. Show the code on the phone

The output now has `First copy your one-time code: XXXX-XXXX`. Show it in the large copy sheet with the page one tap away:

```bash
printf '%s\n' "XXXX-XXXX" | openbase-coder user phone show-text --text-stdin --label "GitHub code" --open "https://github.com/login/device"
```

Tell the user: "Copy the code, tap Open, paste it and approve." If `show-text` cannot reach the phone, open `https://github.com/login/device` with `openbase-coder browser open` and say the code in your reply (and aloud during a call).

Then press Enter in the session, so `gh` stops waiting at `Press Enter to open github.com in your browser...`:

```bash
printf '\n' | openbase-coder pty send gh-login
```

If that opens a second copy of the page, it does no harm.

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

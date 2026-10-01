---
name: openbase-recommended-loops
description: >-
  Use this skill when a user wants a ready-made Openbase loop instead of
  designing one from scratch: a local code reviewer that answers
  `.signals/ready-for-review.md` files, or any other template listed here.
  Each entry is a prompt plus the exact `openbase-coder loops` commands to
  instantiate it on this machine; the product never installs loops on its own.
version: 0.1.0
---

# Openbase Recommended Loops

Openbase ships the generic loop engine (`openbase-loops` skill) and, separately, this library of loop templates. A template is a prompt file under `loops/<name>/prompt.md` next to this skill plus the commands below. Installing one is an explicit action for one machine; nothing here is seeded or enabled automatically, and users adapt the prompt freely.

## How To Instantiate A Template

1. Read the template's prompt and its "Project specifics" placeholder.
2. Write the instantiated prompt to a file the user owns (for example `~/Developer/loops/<name>/prompt.md`), filling in the specifics for this machine: which directories to watch, the base branch convention, the cheap verification commands.
3. Create the loop, then its trigger:

```bash
openbase-coder loops create <name> \
  --prompt "$(cat ~/Developer/loops/<name>/prompt.md)" \
  --time 04:00 --fresh-thread-per-run --cwd <project root> \
  --model <model> --reasoning-effort high
openbase-coder loops add-file-trigger <name> --path '<absolute glob>'
```

The daily `--time` is a catch-up sweep (the prompt handles "no triggering event" by scanning for pending requests); the file trigger does the real work. Re-run `loops create` with the same name after editing the prompt.

## Templates

### `local-review`: file-triggered worktree reviewer

Any agent working in a worktree asks for a second-opinion review by writing `.signals/ready-for-review.md` (a short statement of the task). This loop reviews the diff against that task, fixes what it can, commits on the feature branch, and writes `.signals/review-response.md`. It never pushes, never touches other checkouts, and claims the request with an in-progress response so parallel runs do not collide. Prompt: `loops/local-review/prompt.md`.

Typical trigger, for worktrees that live beside their repos in `*-worktrees/` folders:

```bash
openbase-coder loops add-file-trigger local-review \
  --path '/path/to/*-worktrees/*/.signals/ready-for-review.md' \
  --description "Local review requests"
```

Requesting a review from a worktree:

```bash
mkdir -p .signals && cat > .signals/ready-for-review.md <<'MSG'
---
openbase_signal:
  kind: ready-for-review
  status: open
  from: <agent or thread name>
  created_at: <ISO 8601 UTC>
---

Task: <what this worktree is supposed to accomplish>. Scope: <areas>. Scrutinize: <risky parts>.
MSG
```

Then poll until `.signals/review-response.md` is newer than the request, read it, verify the reviewer's commits, and address the rest.

## Adding A Template

Add `loops/<name>/prompt.md` and a section above with: what it watches, what it does, what it never does, the trigger command, and how to talk to it. Keep prompts platform-neutral (describe steps; name a tool only as an example) and free of any one user's paths or conventions: those go in the "Project specifics" section the installer fills in.

---
name: where-was-i
description: Reconstruct what was being worked on in the current repo — recent commits, reflog moves, unstaged changes, and current branch state. Use at the start of a session to pick up context without the user narrating. Also use when the user says "where was I", "what was I doing", or "pick up where we left off".
---

# Where was I

Dump recent git activity so the agent can reconstruct context without the user re-explaining.

## Steps

Run each of these in the current repo. Report findings grouped by source, noting anything surprising.

1. **Current state**:
   ```bash
   git status --short --branch
   git log -5 --oneline --decorate
   ```

2. **Unstaged + staged diff summary** (filenames only, no body — keeps output small):
   ```bash
   git diff --stat
   git diff --cached --stat
   ```

3. **Recent branch moves** (reflog — where you've been in the last 24h):
   ```bash
   git reflog --date=relative -20
   ```

4. **Stashes** (often forgotten work):
   ```bash
   git stash list
   ```

5. **Worktrees** (may have parallel work elsewhere):
   ```bash
   git worktree list
   ```

6. **Untracked files** (new scaffolding not yet added):
   ```bash
   git ls-files --others --exclude-standard
   ```

## Report

Summarize in this order:
- **Branch + upstream state** — clean / ahead / behind / detached
- **In-progress work** — unstaged/staged file list, any obvious WIP
- **Recent moves** — what branches were touched in the last day
- **Parallel work** — worktrees and stashes if any
- **Best guess** — one sentence: "Looks like you were in the middle of X on branch Y."

Keep it tight — this is context loading, not an audit. Skip sections that returned nothing.

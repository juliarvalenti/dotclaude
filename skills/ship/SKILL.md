---
name: ship
description: Full workflow from in-progress changes to merged PR — branch check, precommit, commit, push, PR, watch checks, merge on approval. Use when the user says "ship", "ship it", "ship this", "PR this", "push this up".
argument-hint: "[optional commit/PR hints]"
user_invocable: true
---

# Ship

Take changes from wherever they are to a merged PR.

## Steps

### 1. Branch check

```bash
git rev-parse --abbrev-ref HEAD
```

If on `main` / `master`, stop and ask for a branch name before proceeding. Infer a sensible name from the changes (`feat/tooltips`, `fix/auth-redirect`) if the user doesn't give one.

```bash
git checkout -b <branch>
```

### 2. Precommit

Run `/precommit` if the project has one. If any step fails, stop and report — don't paper over with `--no-verify`.

### 3. Review and stage

```bash
git status
git diff --stat
```

Stage with specific paths (not `git add -A`) unless the user explicitly says "stage everything":

```bash
git add path/to/file1 path/to/file2
```

**Always flag before staging**: `.env*`, credentials, large binaries, anything in a `secrets/` dir.

### 4. Commit

Use a conventional-commit prefix:
- `feat:` new feature
- `fix:` bug fix
- `refactor:` behavior-preserving rewrite
- `docs:` documentation
- `test:` test changes
- `chore:` dependencies, build config, tooling
- `style:` formatting only

Format via HEREDOC to preserve body lines:

```bash
git commit -m "$(cat <<'EOF'
<type>: <short summary — imperative, no trailing period>

<optional body — why, not what. One paragraph or a few bullets.>
EOF
)"
```

Never amend a published commit. Never `--no-verify` to bypass hooks.

### 5. Push

```bash
git push -u origin <branch>
```

Never force-push to main/master. For force-pushing a feature branch: `--force-with-lease`, not `--force`.

### 6. Open PR

```bash
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
- <bullet>
- <bullet>

## Test plan
- [ ] <how to verify this works>
- [ ] <edge cases or regressions to check>
EOF
)"
```

Keep the title under 70 chars — use the body for detail. Match any repo-specific PR template if one exists (`.github/pull_request_template.md`).

### 7. Watch checks

```bash
gh pr checks --watch --fail-fast
```

If checks fail, investigate and fix before proposing merge. Fixes go as new commits on the same branch — don't amend.

### 8. Ask before merging

Show the PR URL and the check status. Ask:
> "Checks passed — ready to merge. Should I go ahead?"

**Do not merge without explicit confirmation.**

### 9. Merge on approval

```bash
gh pr merge --squash --delete-branch
```

Default: squash + delete branch. Only use `--merge` (no squash) if the project convention is preserve-history.

Confirm merge succeeded and branch was cleaned up.

## When to skip this skill

- **No PR workflow** (solo repo, pushing to main directly) → just commit + push, don't invoke this.
- **Hotfix rolling straight to prod** → project may have a dedicated path (see `/deploy` skills, if any).
- **Release tagging** → use `/release` instead.

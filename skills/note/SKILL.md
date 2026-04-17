---
name: note
description: Drop a progress comment on the current branch's PR — a mid-session checkpoint about what's done, what's in progress, or what was decided. Use when the user says "note this", "leave a note on the PR", "checkpoint", "drop a comment on the PR".
user_invocable: true
---

# Note

Post a short progress comment on the PR for the current branch. Use at natural breakpoints — finished a sub-task, hit a decision point, discovered something worth surfacing, parking work for later.

## Steps

1. **Find the PR for the current branch**:
   ```bash
   gh pr view --json number,url,headRefName -q '.number,.url'
   ```
   If there's no PR yet (branch hasn't been pushed, or PR doesn't exist), say so and suggest `/ship` first — don't silently fail.

2. **Compose the note.** Keep it short — a paragraph or a few bullets. The content should be useful for someone reading this PR months from now trying to reconstruct how it evolved.

3. **Post it**:
   ```bash
   gh pr comment <number> --body "$(cat <<'EOF'
   <note content>
   EOF
   )"
   ```

4. **Report** the comment URL so the user can click through.

## Tone — what goes in a note

Notes are not PR descriptions (those live in the PR body and describe the finished product). Notes are checkpoints: the breadcrumbs of work-in-progress.

**Good notes** (factual, specific, useful later):
- "Finished the webhook validator, wired into the POST /events route. Moving on to the queue consumer."
- "Chose to keep the legacy adapter path behind a feature flag rather than removing — existing production data depends on it. See `src/legacy/adapter.ts`."
- "Parking the migration script until the schema PR (#142) lands. Coming back to this after."
- "Discovered the existing `usePollingSession` hook already handles the retry case I was about to build. Dropped the new code, using the hook instead."

**Bad notes** (narrative, vague, low-value):
- "Working on the thing, making good progress!"
- "We decided to use X instead of Y after some discussion."
- "Refactored some stuff."
- "@julia check this out lmk what you think" (Slack-shaped, not documentation-shaped)

## When to use

- Between meaningful sub-tasks in a long PR
- When a decision gets locked in that future-you will want to find
- When you park work and want a pointer for the next session
- When you discover something about existing code that changed the plan

## When not to use

- Every commit — noise
- Right before merging — that's the PR description's job
- To ask for review — use `gh pr review --request` or just @mention in a normal comment
- For TODOs — put them in the code as `// TODO:` with context, or as a follow-up issue

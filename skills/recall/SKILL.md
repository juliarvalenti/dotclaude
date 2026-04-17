---
name: recall
description: Reconstruct context — current repo state ("where was I"), past Claude Code sessions ("what did we decide"), or past PRs ("didn't we already do this"). Use when the user says "where was I", "pick up where we left off", "what did we say about Y", "find the session where we discussed Z", "didn't we close a PR for this", "find the PR that touched X".
user_invocable: true
---

# Recall

Three modes. Match the user's intent:

- **Current state** ("where was I", "what was I doing", "pick up") → [Current repo mode](#current-repo-mode)
- **Past sessions** ("what did we decide", "find the session where", "when did we") → [Session search mode](#session-search-mode)
- **Past PRs** ("didn't we close a PR", "find the PR that touched X", "we did this before") → [PR search mode](#pr-search-mode)

PRs are often the best source for "have we solved this before" — they pair the full diff with a reader-facing description and running commentary. Prefer PR search over session search when the work was likely merged.

---

## Current repo mode

Reconstruct what was being worked on in the current repo without the user narrating.

### Steps

1. **Current state**:
   ```bash
   git status --short --branch
   git log -5 --oneline --decorate
   ```

2. **Diff summary** (filenames only):
   ```bash
   git diff --stat
   git diff --cached --stat
   ```

3. **Recent branch moves** (where you've been in the last day):
   ```bash
   git reflog --date=relative -20
   ```

4. **Stashes + worktrees + untracked** (often-forgotten state):
   ```bash
   git stash list
   git worktree list
   git ls-files --others --exclude-standard
   ```

### Report

In this order, skipping empty sections:
- **Branch + upstream state** — clean / ahead / behind / detached
- **In-progress work** — unstaged/staged file list, any obvious WIP
- **Recent moves** — branches touched in the last day
- **Parallel work** — worktrees, stashes
- **Best guess** — one sentence: "Looks like you were in the middle of X on branch Y."

Keep it tight — context loading, not an audit.

---

## Session search mode

Grep past Claude Code session transcripts.

Sessions live as JSONL at `~/.claude/projects/<slug>/<session-id>.jsonl`, where `<slug>` is the session's cwd with `/` → `-`. Files are large and noisy (tool results, hook output, attachments) — naive `grep` gets buried.

### The pattern

Extract **only user messages and assistant text**, then grep:

```bash
QUERY="your phrase"
SCOPE=~/.claude/projects/<slug>      # project-specific (fast)
# or
SCOPE=~/.claude/projects             # global (slow)

find "$SCOPE" -name "*.jsonl" -print0 \
  | xargs -0 -n1 -P4 sh -c '
      jq -r "select(.type==\"user\" or .type==\"assistant\")
             | select(.message.content != null)
             | [.timestamp, .type, (.message.content
                 | if type==\"array\" then map(select(.type==\"text\")|.text)|join(\" \")
                   else . end)]
             | @tsv" "$1" 2>/dev/null \
        | rg -i --color=never "'"$QUERY"'" \
        | sed "s|^|$1\t|"
    ' _ \
  | head -50
```

Output columns: `path<TAB>timestamp<TAB>role<TAB>matching-text`.

### Scope decision

- If the user's question is about a specific project → derive slug from that project's path, search only that dir
- If global ("have we ever talked about X") → `~/.claude/projects`

### Narrow by date

Add after the jq pipeline if a timeframe is given:
```bash
| awk -F'\t' '$2 >= "2026-04-01"'
```

### Report

For each match (cap ~5 unless asked):
- **Project**: `~/Documents/GitHub/<project>` (resolve slug → path)
- **When**: relative time (e.g. "3 days ago")
- **Session**: last 8 chars of session ID
- **Snippet**: matching line with ~100 chars of context
- **Role**: user / assistant

Offer to read the full session or a broader excerpt.

### Tips

- `isSidechain:true` events are subagent runs — usually skip for "what did I say" queries
- User-role filter is gold when the question is "what did I say about X"
- For repeat searches in one conversation, consider dumping `{timestamp, type, text}` to a sidecar file the first time, then grep those — cheap cache

---

## PR search mode

Past PRs are often the best source for "have we solved this before." Body + comments + diff live together under a human-edited description.

### Repo-scoped search

When the user is in a repo and asking about its history:

```bash
gh pr list --state all --search "<query>" --limit 10 \
  --json number,title,state,mergedAt,headRefName,url \
  --template '{{range .}}{{.number}}	{{.state}}	{{.mergedAt | timeago}}	{{.title}}	{{.url}}{{"\n"}}{{end}}'
```

**Query tips:**
- `in:title <term>` — search titles only
- `author:@me` — PRs you opened
- `is:merged` / `is:closed` / `is:open`
- `<filename>` — GitHub search indexes changed paths, so `src/lib/client.ts` finds PRs that touched that file
- `<keyword>` alone searches title + body + comments

### Cross-repo search (user-wide)

When the question spans multiple repos ("have I ever worked on auth redirects"):

```bash
gh search prs "<query>" --author=@me --limit 10 \
  --json number,title,repository,url,state \
  --template '{{range .}}{{.repository.nameWithOwner}}#{{.number}}	{{.state}}	{{.title}}	{{.url}}{{"\n"}}{{end}}'
```

### Read a specific PR in full

```bash
gh pr view <number> --comments
gh pr diff <number>
```

`--comments` includes the description plus every top-level comment and review comment. The diff is attached separately — you can pipe it through `grep`, `head`, or feed specific files.

For just the description:
```bash
gh pr view <number> --json title,body,mergedAt,url -q '{title, body, mergedAt, url}'
```

### Report

For each hit:
- **Repo#PR**: `owner/repo#123`
- **State**: merged / closed / open
- **When**: relative (`merged 3 days ago`)
- **Title** and **URL**
- **One-line summary** of relevance — pull a matching line from the body or comment if a query term appears there, so the user sees *why* this PR was returned.

Offer to pull the full description, comments, or diff for any of the hits.

### When to reach for PR search vs session search

- **Merged work** → PR search (cleaner signal, includes the final state + rationale)
- **Unmerged exploration, things we ruled out, half-built ideas** → session search
- **"What did *I* say about X" (my reasoning, not code)** → session search with user-role filter
- **"Where did we build X", "how did we solve Y"** → PR search

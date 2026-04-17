---
name: recall
description: Reconstruct context — either current repo state ("where was I") or by searching past Claude Code sessions ("what did we decide about X"). Use when the user says "where was I", "pick up where we left off", "what did we say about Y", "find the session where we discussed Z".
user_invocable: true
---

# Recall

Two modes. Match the user's intent:

- **Current state** ("where was I", "what was I doing", "pick up") → [Current repo mode](#current-repo-mode)
- **Past sessions** ("what did we decide", "find the session where", "when did we") → [Session search mode](#session-search-mode)

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

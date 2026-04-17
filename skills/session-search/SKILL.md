---
name: session-search
description: Search past Claude Code sessions for a keyword or phrase. Use when the user asks "what did we decide about X?", "when did I last work on Y?", "find the session where we discussed Z", or wants to recall prior context across sessions. Returns matching snippets with session ID, timestamp, and project path so the user can jump back.
---

# Session Search

Search past Claude Code session transcripts. Sessions live as JSONL files under `~/.claude/projects/<slug>/<session-id>.jsonl`, where `<slug>` is the session's cwd with `/` replaced by `-`.

## Why this is tricky

Session JSONL files are large, noisy, and include:
- Tool results (can be huge — file dumps, command output)
- Hook stdout/stderr
- System reminders and attachments
- Embedded base64 for images

Plain `grep` returns matches in all of this. You want **user messages** and **assistant text** — not a hit inside a 50KB tool result or a hook log.

## Steps

1. **Decide the scope.** Project-specific search is much faster than global.
   - Current project: slug = cwd with `/` → `-`, e.g. `/Users/juliavalenti/Documents/GitHub/rnz` → `-Users-juliavalenti-Documents-GitHub-rnz`
   - Global: search all projects under `~/.claude/projects/`

2. **Extract text-only events with jq, then grep.** This pattern is orders of magnitude cheaper than raw grepping the JSONL:

   ```bash
   QUERY="your search phrase"
   SCOPE=~/.claude/projects/<slug>   # or ~/.claude/projects for global

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

3. **Narrow by date** if the user gives a rough timeframe. Add after the jq pipeline:
   ```bash
   | awk -F'\t' '$2 >= "2026-04-01"'
   ```

4. **Pull full context for the best hit.** Once you've identified the interesting session(s), read ~20 lines around the match from the JSONL directly. Use the timestamp to find the line, then `jq` that line for full content.

5. **Resolve slug → project path** when reporting, so the user sees `~/Documents/GitHub/rnz` not `-Users-juliavalenti-Documents-GitHub-rnz`.

## Report

For each match (max ~5 unless asked for more):
- **Project**: `~/Documents/GitHub/<project>`
- **When**: relative time (e.g. "3 days ago")
- **Session**: last 8 chars of session ID
- **Snippet**: the matching line, ~100 chars of context
- **Role**: user / assistant

Offer to open the full session (`~/.claude/projects/<slug>/<session-id>.jsonl`) or extract a broader excerpt.

## Tips

- **`--type=user` filter** is gold when you want "things the user said" — cuts out assistant verbosity.
- **Avoid loading whole files into context.** Use `head`, `rg --max-count`, or line-range reads.
- **Sessions with `isSidechain:true`** are subagent runs — often worth skipping for "what did I say" queries.
- **Cache extracted text** if the user searches frequently: dump `{timestamp, type, text}` per session to a sidecar file the first time, then grep those. Only do this if searches happen repeatedly in one conversation.

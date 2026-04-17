---
name: handoff
description: Assemble a compact context packet you can paste into a fresh Claude Code session to pick up where the current one left off. Use when the user says "hand off", "pack up", "context dump for new session", "summarize and give me something to paste".
user_invocable: true
---

# Handoff

Produce a markdown packet that captures just enough context for a fresh session to continue the work. The goal is density, not completeness — the new session will explore the code itself.

## Output shape

The final deliverable is a single markdown block, copied to the clipboard via `pbcopy`, containing the sections below. Keep the whole thing under ~400 lines — it's paste-bait, not a manuscript.

```markdown
# Handoff — <project>

## Where I am
- Repo: <abs path>
- Branch: <branch> (<clean|N ahead|N behind|dirty>)
- Upstream: <origin/branch or "no upstream">

## What I was doing
<1-3 sentences — the user's current objective. Pull from their most recent messages.>

## Recent decisions
<3-7 bullets — non-obvious choices made this session that future-me should know.
Skip routine stuff. Include why, not just what.>

## In-progress changes
<file-level summary of staged + unstaged work. Group by area if many files.>

## Open threads
<Questions asked but not answered, things parked for later, known broken states.>

## Next step
<One sentence: the very next action.>
```

## Steps

1. **Git state**:
   ```bash
   pwd
   git status --short --branch
   git diff --stat
   git diff --cached --stat
   git log -3 --oneline
   ```

2. **Find the current session transcript** to mine for "what the user said":
   ```bash
   SLUG=$(pwd | sed 's|/|-|g')
   SESSION=$(ls -t ~/.claude/projects/$SLUG/*.jsonl 2>/dev/null | head -1)
   ```
   The most recently modified JSONL in the cwd's slug dir is this session.

3. **Extract the user's prompts** from the session (skip tool results / attachments):
   ```bash
   jq -r 'select(.type=="user" and (.message.content | type) == "string") | .message.content' "$SESSION" | tail -20
   ```
   These are the user's own words — the best source for "what I was doing."

4. **Synthesize**. Read steps 1-3, then write the markdown packet above. Pull from your own memory of the session for "Recent decisions" and "Open threads" — those need judgment, not grep.

5. **Copy to clipboard**:
   ```bash
   pbcopy <<'EOF'
   <rendered markdown>
   EOF
   ```
   Confirm with one sentence ("Handoff on clipboard — ~X lines").

## Principles

- **Terse beats complete.** The new session will re-derive details by reading code.
- **Quote the user's own phrasing** where possible in "What I was doing" — preserves intent better than paraphrase.
- **Skip project conventions** (CLAUDE.md handles those). Don't re-explain the codebase.
- **Name the next step concretely.** "Finish the route handler" is vague; "Add PATCH /api/interviews/[id] with ownership check like GET" is actionable.
- **Flag what's broken.** If tests are failing or a dep is missing, say so up front — saves the next session from re-discovering.

## When not to use

- Conversation is ≤ 5 turns — just tell the new session what you want directly.
- No code changes happened — nothing to hand off.

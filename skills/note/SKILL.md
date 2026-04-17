---
name: note
description: Drop a durable note — either a PR comment (in-progress checkpoint) or a wiki page (long-form context that outlives the PR). Use when the user says "note this", "leave a note", "checkpoint", "drop a comment on the PR", "add this to the wiki", "wiki this".
user_invocable: true
---

# Note

Write a durable note into GitHub. Two destinations, same skill:

- **PR comment** — mid-session checkpoints. Short, factual, tied to the work happening right now. Lives on the PR forever as the archaeology of how it came together.
- **Wiki page** — long-form reference that outlives any one PR. Design rationale, concept notes, onboarding material, anything you'd want to find months later without digging through merged PRs.

Karpathy framing: the markdown files *are* the knowledge base. Unlike Karpathy's autonomous version, here the user is editor-in-chief — the agent drafts, the user curates. The public-under-my-name forcing function keeps quality high.

## Pick the destination

Default based on content shape:
- **In-progress, short, tied to current work** → PR comment
- **Conceptual, long-form, not tied to a single PR** → wiki page
- **User explicitly says "wiki" / "comment on PR"** → honor the explicit choice

If ambiguous, ask. Don't silently default to the wrong surface.

---

## Mode 1 — PR comment

### Steps

1. **Find the PR for the current branch**:
   ```bash
   gh pr view --json number,url,headRefName -q '.number,.url'
   ```
   If no PR exists, say so and suggest `/ship` first — don't silently fail.

2. **Compose the note** — a paragraph or a few bullets. Content should help a future reader reconstruct how the PR evolved.

3. **Post it**:
   ```bash
   gh pr comment <number> --body "$(cat <<'EOF'
   <note content>
   EOF
   )"
   ```

4. **Report** the comment URL so the user can click through.

### Tone — what goes in a PR comment

Comments are checkpoints: breadcrumbs of work-in-progress. Not the PR description (that's the finished artifact — see `/ship`).

**Good** (factual, specific, useful later):
- "Finished the webhook validator, wired into the POST /events route. Moving on to the queue consumer."
- "Chose to keep the legacy adapter path behind a feature flag rather than removing — existing production data depends on it. See `src/legacy/adapter.ts`."
- "Parking the migration script until the schema PR (#142) lands. Coming back to this after."
- "Discovered the existing `usePollingSession` hook already handles the retry case I was about to build. Dropped the new code, using the hook instead."

**Bad** (narrative, vague, low-value):
- "Working on the thing, making good progress!"
- "We decided to use X instead of Y after some discussion."
- "Refactored some stuff."
- "@julia check this out lmk what you think" (Slack-shaped, not documentation-shaped)

### When to use

- Between meaningful sub-tasks in a long PR
- When a decision gets locked in that future-you will want to find
- When you park work and want a pointer for the next session
- When you discover something about existing code that changed the plan

### When not to use

- Every commit — noise
- Right before merging — that's the PR description's job
- To ask for review — use `gh pr review --request` or just @mention in a normal comment
- For TODOs — put them in the code as `// TODO:` with context, or as a follow-up issue

---

## Mode 2 — Wiki page

GitHub wikis are separate git repos at `<repo>.wiki.git`. Workflow: clone (or reuse a clone), edit markdown, commit, push.

### Steps

1. **Resolve the wiki repo for the current project**:
   ```bash
   REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
   WIKI_URL="https://github.com/${REPO}.wiki.git"
   WIKI_DIR="/tmp/$(basename $REPO).wiki"
   ```

2. **Clone or update** the wiki locally:
   ```bash
   if [ -d "$WIKI_DIR" ]; then
     git -C "$WIKI_DIR" pull --quiet
   else
     git clone --quiet "$WIKI_URL" "$WIKI_DIR"
   fi
   ```
   If the clone fails with "Repository not found", the wiki hasn't been initialized — tell the user to create the first page via the GitHub web UI at `https://github.com/${REPO}/wiki`, then retry.

3. **Decide: new page or append**. Work with the user to pick a page name (`Design-decisions`, `Auth-flow`, `Onboarding`, etc. — hyphens become spaces in GitHub's UI). If a relevant page exists, append a new dated section. If not, create it.

4. **Write the markdown**:
   ```bash
   # New page
   cat > "$WIKI_DIR/${PAGE}.md" <<'EOF'
   # Page title

   <content>
   EOF

   # Or append a dated section to an existing page
   cat >> "$WIKI_DIR/${PAGE}.md" <<EOF

   ## $(date +%Y-%m-%d) — <short heading>

   <content>
   EOF
   ```

5. **Commit and push**:
   ```bash
   git -C "$WIKI_DIR" add "${PAGE}.md"
   git -C "$WIKI_DIR" -c commit.gpgsign=false commit -q -m "note: <short summary>"
   git -C "$WIKI_DIR" push --quiet
   ```

6. **Report** the URL: `https://github.com/${REPO}/wiki/${PAGE}`.

### Tone — what goes on a wiki page

Wiki content is the long-form companion to code. It explains *why* at a scope larger than any one PR. Written for someone who isn't in the session.

**Good content for a wiki page**:
- Architecture notes ("Why we use Postgres LISTEN/NOTIFY for real-time instead of WebSockets")
- Onboarding / setup walkthroughs the README is too short for
- Concept pages ("What a 'room' is and how it differs from a 'session'")
- Cross-cutting design decisions that touch multiple PRs
- Interview-style summaries of a domain ("How competitive VGC team building works")

**Not for the wiki**:
- Anything derivable from the code itself (architecture that's obvious from the module layout, API shapes that live in route handlers)
- In-progress status — that's a PR comment
- Living specs that should be committed alongside the code — put in `docs/` instead

### Link back from the code

If a wiki page documents something specific to a module, drop a short comment near the top of the module:

```ts
// Design notes: https://github.com/<owner>/<repo>/wiki/<Page>
```

This is the discoverability trick — wikis are invisible if nothing points to them.

### Compared to Karpathy's LLM Wiki

Karpathy's version has the LLM autonomously building and maintaining an Obsidian vault — the agent is the librarian. Here, the agent is a scribe and the user is the editor. The public-under-my-name forcing function is doing real work: Julia edits what goes in, which keeps the wiki from bloating into AI slurry.

Don't try to auto-maintain pages here. One note per invocation, user-reviewed before push.

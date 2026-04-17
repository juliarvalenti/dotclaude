---
name: note
description: Drop a durable note — either a PR comment (in-progress checkpoint) or a GitHub Discussion (long-form context, tagged `ai-drafted` for later curation). Use when the user says "note this", "leave a note", "checkpoint", "drop a comment on the PR", "discussion about X", "write this up".
user_invocable: true
---

# Note

Write a durable note into GitHub. Two destinations, same skill:

- **PR comment** — mid-session checkpoints. Short, factual, tied to current work. Lives on the PR forever as the archaeology of how it came together.
- **Discussion** — shared, persistent knowledge for the team. Long-form context that outlives any one PR: design rationale, architecture notes, concept pages, onboarding material — anything that should be findable by anyone on the project, months later, without spelunking through merged PRs.

## Why Discussions, not wiki

Surface contracts matter as much as content. A wiki implies "ground truth" — readers expect accuracy, and AI-drafted material on that surface over-claims. A Discussion implies "conversational, may age" — readers calibrate skepticism appropriately, and AI-drafted content fits the surface's natural contract.

Every Discussion created by this skill is tagged `ai-drafted` so provenance is explicit. The user-as-editor-in-chief pattern: the agent drafts, the human curates. Remove the label (or promote to `ai-reviewed`) once you've audited. The label never lies about where the content came from.

## Pick the destination

Default based on content shape:
- **In-progress, short, tied to current work** → PR comment
- **Conceptual, long-form, not tied to a single PR** → Discussion
- **User explicitly says "discussion" / "comment on PR"** → honor the choice

If ambiguous, ask. Don't silently default to the wrong surface.

---

## Mode 1 — PR comment

### Steps

1. **Find the PR for the current branch**:
   ```bash
   gh pr view --json number,url,headRefName -q '.number,.url'
   ```
   If no PR exists, say so and suggest `/ship` first — don't silently fail.

2. **Compose** — a paragraph or a few bullets. Should help a future reader reconstruct how the PR evolved.

3. **Post**:
   ```bash
   gh pr comment <number> --body "$(cat <<'EOF'
   <note content>
   EOF
   )"
   ```

4. **Report** the comment URL.

### Tone — PR comments

Checkpoints: breadcrumbs of work-in-progress. Not the PR description (that's the finished artifact — see `/ship`).

**Good** (factual, specific, useful later):
- "Finished the webhook validator, wired into POST /events. Moving to the queue consumer."
- "Chose to keep the legacy adapter path behind a flag — existing production data depends on it. See `src/legacy/adapter.ts`."
- "Parking the migration script until schema PR #142 lands."
- "Discovered `usePollingSession` already handles the retry case I was about to build. Using the hook instead."

**Bad** (narrative, vague, low-value):
- "Working on the thing, making good progress!"
- "We decided to use X instead of Y after some discussion."
- "Refactored some stuff."
- "@julia check this out lmk what you think" (Slack-shaped, not documentation-shaped)

### When to use (PR mode)

- Between meaningful sub-tasks in a long PR
- When a decision gets locked in that future-you will want to find
- When you park work and want a pointer for the next session
- When you discover something about existing code that changed the plan

### When not to use (PR mode)

- Every commit
- Right before merging (that's the PR description's job)
- To request review (`gh pr review --request` or @mention)
- For TODOs (put them in code as `// TODO:` or a follow-up issue)

---

## Mode 2 — Discussion

### Steps

1. **Resolve the repo** and confirm Discussions are enabled:
   ```bash
   REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
   gh api "repos/${REPO}" --jq '.has_discussions'
   ```
   If `false`: tell the user to enable Discussions (Settings → Features → Discussions) and create at least one category. Don't try to enable it automatically.

2. **Find the target category ID.** Ask the user which category (typical names: `General`, `Ideas`, `Notes`, `Design`, `Architecture`). List what's available:
   ```bash
   gh api graphql -f query='
     query($owner:String!, $name:String!) {
       repository(owner:$owner, name:$name) {
         discussionCategories(first:20) { nodes { id name emoji } }
       }
     }' -F owner="${REPO%/*}" -F name="${REPO#*/}" \
     --jq '.data.repository.discussionCategories.nodes[] | "\(.name)\t\(.id)"'
   ```

3. **Get the repository ID** (needed for creation):
   ```bash
   REPO_ID=$(gh api graphql -f query='
     query($owner:String!, $name:String!) {
       repository(owner:$owner, name:$name) { id }
     }' -F owner="${REPO%/*}" -F name="${REPO#*/}" --jq '.data.repository.id')
   ```

4. **Create the discussion**:
   ```bash
   RESULT=$(gh api graphql -f query='
     mutation($repo:ID!, $cat:ID!, $title:String!, $body:String!) {
       createDiscussion(input:{
         repositoryId:$repo, categoryId:$cat, title:$title, body:$body
       }) { discussion { id number url } }
     }' -F repo="$REPO_ID" -F cat="$CATEGORY_ID" -F title="$TITLE" -F body="$BODY")

   NUMBER=$(echo "$RESULT" | jq -r '.data.createDiscussion.discussion.number')
   URL=$(echo "$RESULT" | jq -r '.data.createDiscussion.discussion.url')
   ```

5. **Apply the `ai-drafted` label**. Labels work across Issues, PRs, and Discussions via the REST API:
   ```bash
   # Create the label if it doesn't exist (one-time, idempotent via `|| true`)
   gh api repos/${REPO}/labels -f name='ai-drafted' -f color='9b6dff' -f description='Drafted by an LLM, pending human review' 2>/dev/null || true

   # Apply to the discussion
   gh api repos/${REPO}/issues/${NUMBER}/labels -f labels='["ai-drafted"]'
   ```
   (Discussions use the shared `/issues/<number>/labels` endpoint — GitHub unified the label system.)

6. **Report** the URL. Remind the user: the `ai-drafted` label stays on until *they* remove it or swap it for `ai-reviewed`. The skill never auto-promotes.

### Tone — Discussions

Long-form companion to code. Explains *why* at a scope larger than any one PR. Written for someone who isn't in the session.

**Good content**:
- Architecture notes ("Why Postgres LISTEN/NOTIFY for real-time instead of WebSockets")
- Concept pages ("What a 'room' is and how it differs from a 'session'")
- Onboarding / setup walkthroughs the README is too short for
- Cross-cutting design decisions that touch multiple PRs
- Interview-style summaries of a domain

**Not for a Discussion**:
- Anything derivable from the code itself
- In-progress status — that's a PR comment
- Living specs that should be committed alongside the code — put in `docs/` instead

### Link back from the code

If a Discussion documents something specific to a module, drop a short comment near the top of that module:

```ts
// See: https://github.com/<owner>/<repo>/discussions/<N>
```

Discussions are invisible if nothing points to them.

### Compared to Karpathy's LLM Wiki

Karpathy's version has the LLM autonomously maintaining an Obsidian vault — the agent is librarian. Here the agent is scribe, the user is editor, and the surface itself (Discussions, not wiki) calibrates reader expectations for AI-drafted content. The `ai-drafted` label makes provenance impossible to miss.

Don't auto-maintain. One note per invocation, user-reviewed before the label comes off.

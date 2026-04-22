```
      .o8                .             oooo                              .o8
     "888              .o8             `888                             "888
 .oooo888   .ooooo.  .o888oo  .ooooo.   888   .oooo.   oooo  oooo   .oooo888   .ooooo.
d88' `888  d88' `88b   888   d88' `"Y8  888  `P  )88b  `888  `888  d88' `888  d88' `88b
888   888  888   888   888   888        888   .oP"888   888   888  888   888  888ooo888
888   888  888   888   888 . 888   .o8  888  d8(  888   888   888  888   888  888    .o
`Y8bod88P" `Y8bod8P'   "888" `Y8bod8P' o888o `Y888""8o  `V88V"V8P' `Y8bod88P" `Y8bod8P'
```

My personal boilerplate for Claude Code: the `CLAUDE.md` shape I reach for, the skills I reuse across projects, and a style checklist I run new repos against.

Not a framework. Not opinions for anyone else. This is scaffolding I copy into new projects so I stop rewriting the same structure from scratch.

## Contents

- **[`CLAUDE.template.md`](CLAUDE.template.md)** — blank CLAUDE.md with my canonical sections and inline notes on what goes where
- **[`STYLE.md`](STYLE.md)** — the 12 patterns I actually use, with "when to use it" and "when to skip it"

### Boilerplate skills

- **[`skills/precommit/`](skills/precommit/SKILL.md)** — generic precommit skill to fork per project
- **[`skills/modern-python/`](skills/modern-python/SKILL.md)** — uv / ruff / pytest practices. Upstream from [trailofbits/skills](https://github.com/trailofbits/skills), CC BY-SA 4.0 (see skill's `ATTRIBUTION.md`)
- **[`skills/modern-typescript/`](skills/modern-typescript/SKILL.md)** — pnpm / turbo / vitest / eslint + tsconfig templates, Next.js route+hook co-location

### Utility skills (slash-command UX)

- **[`skills/grab/`](skills/grab/SKILL.md)** — read/write clipboard, open last download, view last screenshot
- **[`skills/recall/`](skills/recall/SKILL.md)** — reconstruct context (current git state, past sessions, or past PRs)
- **[`skills/handoff/`](skills/handoff/SKILL.md)** — pack a context dump for a fresh session
- **[`skills/net/`](skills/net/SKILL.md)** — ports (show / kill / all listening), local + public IP, DNS lookup
- **[`skills/disk/`](skills/disk/SKILL.md)** — observational disk usage: free space, biggest dirs, Docker disk, drill-down
- **[`skills/notify/`](skills/notify/SKILL.md)** — send a macOS notification from the agent (task done, input needed)
- **[`skills/caffeinate/`](skills/caffeinate/SKILL.md)** — keep the prompt cache warm while stepping away (4-min `/loop` ping, beats the 5-min TTL rewrite)
- **[`skills/docker/`](skills/docker/SKILL.md)** — containers + compose (list, logs, shell, prune, up/down)
- **[`skills/pm2/`](skills/pm2/SKILL.md)** — PM2 process ops (list, logs, restart, stop, delete), local or over SSH
- **[`skills/ship/`](skills/ship/SKILL.md)** — full flow from changes to merged PR (branch → precommit → commit → PR → watch → merge)
- **[`skills/note/`](skills/note/SKILL.md)** — drop a durable note: PR comment (in-progress) or GitHub Discussion (long-form, auto-tagged `ai-drafted`)
- **[`skills/release/`](skills/release/SKILL.md)** — tag + changelog + GitHub release (patch/minor/major bump)

## How I use this

**New project:**
1. Copy `CLAUDE.template.md` → `<repo>/CLAUDE.md`, fill in the blanks, delete sections that don't apply
2. Cross-check against `STYLE.md`
3. Copy `skills/precommit/` into `<repo>/.claude/skills/` and customize the commands

**Auditing an existing project:** run through `STYLE.md` and see which sections are missing or stale.

**Adding a new skill to this repo:** drop it in `skills/<name>/SKILL.md`, then add a one-line bullet for it under the matching section above (boilerplate / utility). The README is the index — a skill that isn't listed here is invisible.

## Principles

- **Concrete over abstract.** Real file paths, real commands, never "it depends."
- **No speculative sections.** If the project doesn't have a gotcha list yet, don't add an empty heading.
- **Terse.** CLAUDE.md is read by an agent on every turn — every line should pay rent.
- **Project-specific, not shared.** I intentionally don't merge rules across projects. Each CLAUDE.md is tuned to one codebase.

## What this isn't

I intentionally avoid persona skills — no `pr-reviewer`, no `code-writer`, no `research-partner`, no "you are an expert X who always Y." Those patterns ask the agent to play a character, which bloats context and doesn't compose. This repo treats the skill system as **macros**: shortcuts I can invoke mid-flow to request a specific action, load only the context I need, and manipulate the agent's behavior in the moment. Aimed at pair-coding, not autonomous agents.

## Context lives on GitHub

Context management, design history, and in-progress status all live on GitHub — not in separate planning docs, not in a sidecar memory system, not in bespoke spec files. The tooling here is shaped around that.

- **PRs hold design decisions.** The description is the artifact: reader-facing, under my name, searchable via `gh cli`. `/ship` enforces the "finished artifact, not session diary" tone so this durability actually pays off.
- **PR comments hold in-progress status.** Checkpoints, parked work, discoveries mid-build. `/note` drops them at natural breakpoints so future-me (or a coworker) can reconstruct how a PR evolved.
- **Wikis hold conversations.** Long-form discussion, architecture notes, onboarding, anything that outlives a single PR and doesn't want to live in the code.

`/recall` searches across all three (plus past Claude Code sessions). If the context exists, it's reachable with `gh`. No special index to maintain, no drift between docs and code — just the public trail of the work.

This is an explicitly developer-shaped workflow. If you're not working on GitHub, most of what's here won't map.

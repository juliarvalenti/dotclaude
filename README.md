# dotclaude

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
- **[`skills/recall/`](skills/recall/SKILL.md)** — reconstruct context (current git state, or search past sessions)
- **[`skills/handoff/`](skills/handoff/SKILL.md)** — pack a context dump for a fresh session
- **[`skills/kill-port/`](skills/kill-port/SKILL.md)** — free a TCP port
- **[`skills/docker/`](skills/docker/SKILL.md)** — containers + compose (list, logs, shell, prune, up/down)
- **[`skills/pm2/`](skills/pm2/SKILL.md)** — PM2 process ops (list, logs, restart, stop, delete), local or over SSH
- **[`skills/ship/`](skills/ship/SKILL.md)** — full flow from changes to merged PR (branch → precommit → commit → PR → watch → merge)
- **[`skills/release/`](skills/release/SKILL.md)** — tag + changelog + GitHub release (patch/minor/major bump)

## How I use this

**New project:**
1. Copy `CLAUDE.template.md` → `<repo>/CLAUDE.md`, fill in the blanks, delete sections that don't apply
2. Cross-check against `STYLE.md`
3. Copy `skills/precommit/` into `<repo>/.claude/skills/` and customize the commands

**Auditing an existing project:** run through `STYLE.md` and see which sections are missing or stale.

## Principles

- **Concrete over abstract.** Real file paths, real commands, never "it depends."
- **No speculative sections.** If the project doesn't have a gotcha list yet, don't add an empty heading.
- **Terse.** CLAUDE.md is read by an agent on every turn — every line should pay rent.
- **Project-specific, not shared.** I intentionally don't merge rules across projects. Each CLAUDE.md is tuned to one codebase.

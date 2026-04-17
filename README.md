# dotclaude

My personal boilerplate for Claude Code: the `CLAUDE.md` shape I reach for, the skills I reuse across projects, and a style checklist I run new repos against.

Not a framework. Not opinions for anyone else. This is scaffolding I copy into new projects so I stop rewriting the same structure from scratch.

## Contents

- **[`CLAUDE.template.md`](CLAUDE.template.md)** — blank CLAUDE.md with my canonical sections and inline notes on what goes where
- **[`STYLE.md`](STYLE.md)** — the 12 patterns I actually use, with "when to use it" and "when to skip it"
- **[`skills/precommit/`](skills/precommit/SKILL.md)** — generic precommit skill to fork per project
- **[`skills/last-screenshot/`](skills/last-screenshot/SKILL.md)** — view the most recent screenshot
- **[`skills/where-was-i/`](skills/where-was-i/SKILL.md)** — reconstruct context from git state at the start of a session
- **[`skills/session-search/`](skills/session-search/SKILL.md)** — grep past Claude Code sessions efficiently (skips noisy tool results / hooks)
- **[`skills/modern-python/`](skills/modern-python/SKILL.md)** — uv / ruff / pytest practices. Upstream from [trailofbits/skills](https://github.com/trailofbits/skills), CC BY-SA 4.0 (see skill's `ATTRIBUTION.md`)
- **[`skills/modern-typescript/`](skills/modern-typescript/SKILL.md)** — pnpm / turbo / vitest / eslint + tsconfig templates, Next.js route+hook co-location

### Utility skills (slash-command UX)

- **[`skills/clipboard/`](skills/clipboard/SKILL.md)** — read/write the macOS clipboard
- **[`skills/last-download/`](skills/last-download/SKILL.md)** — read the most recent file in `~/Downloads/`
- **[`skills/kill-port/`](skills/kill-port/SKILL.md)** — free a TCP port
- **[`skills/agent-handoff/`](skills/agent-handoff/SKILL.md)** — pack a context dump for a fresh session
- **[`skills/docker-ps/`](skills/docker-ps/SKILL.md)**, **[`docker-logs/`](skills/docker-logs/SKILL.md)**, **[`docker-shell/`](skills/docker-shell/SKILL.md)**, **[`docker-clean/`](skills/docker-clean/SKILL.md)** — container ops
- **[`skills/compose-up/`](skills/compose-up/SKILL.md)**, **[`compose-down/`](skills/compose-down/SKILL.md)** — find and run the nearest `docker-compose.yml`
- **[`skills/pm2-list/`](skills/pm2-list/SKILL.md)**, **[`pm2-logs/`](skills/pm2-logs/SKILL.md)**, **[`pm2-restart/`](skills/pm2-restart/SKILL.md)**, **[`pm2-stop/`](skills/pm2-stop/SKILL.md)** — PM2 process ops (local or over SSH)

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

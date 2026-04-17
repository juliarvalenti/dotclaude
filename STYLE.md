# CLAUDE.md style checklist

Patterns I use. Each has a "use when" and "skip when" — not every section belongs in every repo.

## Structural patterns

### 1. Opening tagline
One sentence: what it is, who it's for, what stack.
- **Use when**: always.
- **Skip when**: never.

### 2. Tone-setting preamble
Second-person instructions that shape the agent's voice ("You are ___. Be ___.").
- **Use when**: the repo's job is to *be* an assistant (team builders, research helpers, domain tutors).
- **Skip when**: it's a normal code project. The agent doesn't need a persona to fix a bug.

### 3. Quick reference / Commands block
Bash block near the top with install / dev / test / lint / build.
- **Use when**: there are runnable commands. (Almost always.)
- **Skip when**: it's a pure-docs or config repo with nothing to run.

### 4. Architecture block
File tree with one-line descriptions per file, optionally followed by a data-flow paragraph.
- **Use when**: more than ~5 source files, or non-obvious wiring between them.
- **Skip when**: tiny repo where reading the files is faster than reading the map.

### 5. Conventions section
Bullet list: package manager, language version, imports, naming, logging, house rules.
- **Use when**: the project has opinions that aren't enforced by tooling.
- **Skip when**: ruff/prettier/eslint cover everything — don't restate them.

### 6. "Adding a new X" recipes
Numbered steps, real file paths. One recipe per common extension point.
- **Use when**: there's a pattern that repeats (commands, routes, themes, handlers).
- **Skip when**: nothing repeats yet. Don't pre-invent extension points.

### 7. Gotchas / "learned the hard way"
Short list of real foot-guns with one-line explanations.
- **Use when**: something has actually bitten you or the agent.
- **Skip when**: you'd be making it up. Empty gotcha sections are worse than no section.

### 8. Skill references
Offload procedures to `@.claude/skills/foo/SKILL.md` and link from CLAUDE.md.
- **Use when**: a procedure is long, used occasionally, or has its own flags/options (deploy, pull-secrets, precommit).
- **Skip when**: the procedure is two lines — just inline it.

### 9. Don't list
❌ bullet list of hard rules with short reasons.
- **Use when**: there are repeat mistakes the agent keeps making, or prescriptive patterns (component shape, auth wrappers).
- **Skip when**: the codebase doesn't have strong rules — don't manufacture them.

### 10. Key files table
Two-column table: "need → path" for the most-searched-for files.
- **Use when**: the repo has 20+ files and agents keep re-searching for the same ones.
- **Skip when**: small repo, or the architecture block already covers it.

## Stylistic patterns

### 11. Concrete over abstract
Every instruction names a real file, real command, or real symbol. No "the relevant module" — say `src/lib/client.ts`.
- **Always apply.**

### 12. Terse
CLAUDE.md is re-read on every turn. Cut anything that doesn't pay rent. No introductory paragraphs, no "this document describes", no polite framing.
- **Always apply.**

## Quick audit

Reading an existing CLAUDE.md, ask:
- Can an agent start work after reading just this? (If no — what's missing?)
- Is anything here derivable from the code itself? (If yes — delete it.)
- Are the gotchas real, or invented? (If invented — delete them.)
- Would I tolerate this length if I had to re-read it every turn?

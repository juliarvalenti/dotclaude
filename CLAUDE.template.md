<!--
  CLAUDE.md template — fill in the {{placeholders}}, delete sections that don't apply.
  Keep it terse. Every section is optional except the opening tagline.
  Inline comments like this one should be stripped from the final file.
-->

# {{Project Name}}

{{One sentence: what it is, who uses it, what stack. Optionally a second sentence on scope or non-goals.}}

<!-- For agent-persona projects (e.g. team builders, research assistants), add a tone-setting paragraph here:
     "You are a ___. The user is ___. Be ___. When you ___, ___."
     See pokemon-champions CLAUDE.md for reference. -->

## Quick reference

```bash
# Install
{{install command}}

# Dev
{{dev command}}

# Test
{{test command}}

# Lint / type-check (what CI runs)
{{lint command}}

# Build
{{build command}}
```

## Architecture

```
{{file tree with one-line descriptions per file — only include files an agent actually needs to know about}}
```

{{Optional: one paragraph describing the signal/data flow through those files.}}

## Conventions

- **Package manager**: {{pnpm | uv | cargo | ...}} — never mix
- **Language version**: {{e.g. Python 3.12+, Node 20+}}
- **Imports**: {{stdlib → third-party → local, path aliases, etc.}}
- **Naming**: {{PascalCase classes, snake_case funcs, etc.}}
- **Logging**: {{per-module logger, no print() in production, etc.}}
- {{Other house rules specific to this repo}}

## Adding a new {{thing}}

<!-- One "how to add X" recipe per common extension point.
     Number the steps. Reference real file paths. Keep under ~5 steps each. -->

1. {{step}}
2. {{step}}
3. {{step}}

## Gotchas

<!-- Things that bit me and will bite the agent. Only add real ones — don't invent. -->

- **{{short name}}** — {{what happens and why, 1-2 lines}}
- **{{short name}}** — {{what happens and why, 1-2 lines}}

## Skills

<!-- Offload project-specific procedures to skills, reference them here.
     The agent loads these lazily — keeps CLAUDE.md itself short. -->

- See `@.claude/skills/{{skill-name}}/SKILL.md` for {{what it does}}

## Don't

<!-- Hard rules. Keep to 5-10. Each should reference a real past mistake or strong preference. -->

- ❌ **Don't** {{thing}} — {{why}}
- ❌ **Don't** {{thing}} — {{why}}

## Key files

| Need | Check |
|------|-------|
| {{thing}} | `{{path}}` |
| {{thing}} | `{{path}}` |

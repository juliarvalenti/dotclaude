---
name: precommit
description: Run precommit checks (lint, format, types, tests) and auto-fix where possible. Use when the user says /precommit or wants to check code quality before committing. Steps are independent — run only the ones that apply to the files changed.
---

# Precommit Checks

<!--
  BOILERPLATE: fork per project. Each step should be independently runnable and
  conditionally skippable so the agent can short-circuit to only the relevant
  checks given the diff. Do NOT wrap everything in one opaque `pnpm precommit`
  script — that prevents targeted runs.

  Replace the steps below with your project's actual services / commands.
-->

Run quality checks on the codebase. Auto-fix issues where possible. Steps are independent — if the diff only touches one service, run only that service's step.

## Steps

1. **Lint + format** — Fix lint and format issues automatically:
   ```bash
   {{e.g. cd service-a && uv run ruff check --fix . && uv run ruff format .}}
   {{cd service-b && pnpm lint --fix && pnpm format}}
   ```

2. **Type check**:
   ```bash
   {{e.g. cd service-a && uv run pyright .}}
   {{cd service-b && npx tsc --noEmit}}
   ```

3. **Tests** — Run the test suite:
   ```bash
   {{e.g. cd service-a && uv run pytest tests/ -x -q}}
   ```

4. **Build** — Verify the production build compiles:
   ```bash
   {{e.g. cd service-b && pnpm build}}
   ```

5. **Conditional: {{subsystem X}}** — If any file under `{{path/to/subsystem}}` was changed, run:
   ```bash
   {{subsystem-specific check}}
   ```
   {{One line on why this exists — what regression it catches.}}

6. **Conditional: generated artifacts** — If `{{schema file / route file / doc source}}` changed, the generated {{client / docs / types}} may be stale. Regenerate:
   ```bash
   {{regeneration command}}
   ```

7. **Conditional: docs consistency** — If any user-facing behavior changed (commands renamed, new features, API changes), grep for stale references and fix them in:
   - `{{README.md}}`
   - `{{docs/...}}`
   - `{{other doc locations}}`

8. **Report** — Summarize what was fixed and any remaining issues. Don't dump full warning output unless the user asks — list categories (e.g. "3 type errors, 12 formatting fixes applied, 1 test failure").

<!--
  Principles for forking this:
  - Every step stands alone. No step depends on a previous step's side effects.
  - Conditional steps name the trigger path so the agent can check `git diff`
    and decide whether to run.
  - Keep the `Report` step last. Always.
-->

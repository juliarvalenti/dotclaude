---
name: modern-typescript
description: Modern TypeScript tooling practices. Use when starting a new TS project, adding deps, configuring tsconfig/eslint/vitest, or reviewing an existing config. Leans Next.js + Prisma for app code and pnpm + turbo for monorepos.
---

# Modern TypeScript Practices

Opinionated baseline for TypeScript projects. pnpm for package management, Vitest for tests, ESLint for linting, no formatter. Framework default: Next.js + Prisma.

## Core Rules

| Do | Don't |
|----|-------|
| `pnpm add <pkg>` | `npm install` / `yarn add` |
| `pnpm add -D <pkg>` | Edit package.json deps manually |
| `pnpm add -w <pkg>` (workspace root) | Install root-only deps inside a workspace package |
| `pnpm dlx <tool>` | `npx <tool>` |
| `pnpm -r run <script>` | Looping packages manually |
| Prisma types directly (`import type { User } from "@/generated/prisma/client"`) | Redefine database entity types |
| Zod for form input + untrusted payloads | Zod for things ORM types already cover |
| Vitest | Jest, Mocha (greenfield projects) |
| ESLint flat config (`eslint.config.mjs`) | Legacy `.eslintrc.json` |
| `"type": "module"` + `"module": "NodeNext"` for libs / CLI | Defaulting to CJS in new code |
| Node version in `.nvmrc` + `engines` | Relying only on pnpm's `packageManager` pin |

## Quick Reference

```bash
# Install
pnpm install                 # uses lockfile
pnpm install --frozen-lockfile   # CI — fails if lockfile drifts

# Add deps
pnpm add zod
pnpm add -D vitest @types/node
pnpm add -w turbo            # add to workspace root

# Scripts
pnpm dev
pnpm build
pnpm test
pnpm lint
pnpm typecheck               # standard alias for `tsc --noEmit`

# Monorepo (via turbo)
pnpm turbo run build
pnpm turbo run test --filter=@scope/pkg

# One-off tool without installing
pnpm dlx tsx script.ts
```

## tsconfig.json baseline

Minimum flags that pay rent. Copy these into every new project.

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022"],
    "module": "NodeNext",
    "moduleResolution": "NodeNext",

    // Strictness — these two alone catch most real bugs
    "strict": true,
    "noUncheckedIndexedAccess": true,

    // Small additions that cost nothing
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,

    // Quality of life
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true,

    // Output
    "outDir": "dist",
    "sourceMap": true,
    "declaration": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

**Adjust per target:**
- **Next.js app**: `"module": "ESNext"`, `"moduleResolution": "Bundler"`, `"lib": ["ES2022", "DOM", "DOM.Iterable"]`, `"jsx": "preserve"`, `"noEmit": true`, path alias `"@/*": ["./src/*"]`. Omit `outDir` / `declaration`.
- **Library**: keep baseline. Add `"composite": true` if part of a workspace with project refs.
- **CLI tool**: keep baseline.

**Skip these flags** (tempting but low value in practice):
- `exactOptionalPropertyTypes` — forces `prop?: T | undefined` distinction, mostly noise
- `noPropertyAccessFromIndexSignature` — pedantic, fights normal code
- `noUnusedLocals` / `noUnusedParameters` — let ESLint handle this (auto-fixable, config-local)

See [`references/tsconfig.md`](references/tsconfig.md) for per-flag explanations.

## ESLint (flat config, no Prettier)

No formatter. Agentic coding makes formatting debates obsolete — if the agent wrote it, the format is whatever the agent picked. Use ESLint for real problems, not whitespace.

```bash
pnpm add -D eslint typescript-eslint
```

Baseline `eslint.config.mjs`:

```js
import tseslint from "typescript-eslint";

export default tseslint.config(
  { ignores: ["dist/**", "node_modules/**", "**/generated/**", "**/.next/**"] },
  ...tseslint.configs.recommended,
  {
    rules: {
      "@typescript-eslint/no-unused-vars": ["warn", { argsIgnorePattern: "^_", varsIgnorePattern: "^_" }],
    },
  },
);
```

**Next.js projects**: also extend `next/core-web-vitals` and `next/typescript` via `@eslint/eslintrc`'s `FlatCompat`. See [`references/eslint.md`](references/eslint.md).

## Vitest

```bash
pnpm add -D vitest
```

Minimal `vitest.config.ts`:

```ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    include: ["src/**/*.test.ts", "tests/**/*.test.ts"],
    environment: "node",  // or "jsdom" for DOM-touching code
  },
});
```

Test script: `"test": "vitest run"`, dev: `"test:watch": "vitest"`.

See [`references/testing.md`](references/testing.md) for patterns (fixtures, mocking, integration tests).

## Monorepo (pnpm + turbo)

Use a monorepo when you have: (a) multiple deployable apps sharing code, (b) a library + its examples, or (c) a frontend + backend that need shared types. Single-package projects should stay single-package.

`pnpm-workspace.yaml`:
```yaml
packages:
  - apps/*
  - packages/*
  - services/*
```

`turbo.json` — see [`references/monorepo.md`](references/monorepo.md) for full patterns. Key rules:
- `build` depends on `^build` (upstream packages build first)
- Output globs include `dist/**`, `.next/**`, `build/**`
- `dev` is `persistent: true`, `cache: false`
- Anything touching secrets or generated code: `cache: false`

Pin `packageManager` in the root `package.json`:
```json
{ "packageManager": "pnpm@9.15.0" }
```

## Validation: Prisma first, Zod as a last resort

**Order of preference:**
1. **Prisma types** for anything that exists in the database schema — `User`, `Session`, enum literals. Import from your generated client.
2. **Derived types** from Prisma using `Prisma.UserGetPayload<{ include: {...} }>` for query-shape types.
3. **Zod** only for data crossing an untrusted boundary: form inputs, webhook payloads, env variables, parsed config files.

**Where Zod schemas live**: centralize per-boundary in `src/types/api/` or `src/schemas/`, not inline in route handlers. One schema per input shape, exported + reused.

**Use `.parse()`, not `.safeParse()`**, inside a try/catch at the route boundary — throw on bad input, catch once, return a user-friendly 400. Don't plumb `Result` types through application code.

```ts
// src/schemas/support-request.ts
export const supportRequestSchema = z.object({ email: z.email(), body: z.string().min(1) });
export type SupportRequest = z.infer<typeof supportRequestSchema>;

// route handler
try {
  const input = supportRequestSchema.parse(await req.json());
  // ... use input (fully typed)
} catch (err) {
  if (err instanceof z.ZodError) return Response.json({ error: err.message }, { status: 400 });
  throw err;
}
```

See [`references/validation.md`](references/validation.md).

## Module system: let the runtime decide

| Project type | Recommendation |
|--------------|----------------|
| Next.js app | Whatever Next wants (bundler handles it) |
| Library | ESM (`"type": "module"`, `"module": "NodeNext"`) — unless consumers need CJS |
| CLI tool | ESM for new projects |
| Node service | ESM unless a critical dep blocks it |
| Legacy / Electron main process / some serverless runtimes | CJS until forced otherwise |

See [`references/module-systems.md`](references/module-systems.md) for the decision tree and known CJS-only deps.

## Node version pinning

Three files, all aligned:

**`.nvmrc`**:
```
22
```

**`package.json`**:
```json
{ "engines": { "node": ">=22" } }
```

**CI workflow** (GitHub Actions):
```yaml
- uses: actions/setup-node@v4
  with:
    node-version-file: .nvmrc
    cache: pnpm
```

The `packageManager` field pins pnpm; `.nvmrc` + `engines` pin Node. Having all three means every environment agrees, and `pnpm install` will warn on mismatch.

## Package.json script conventions

Standard names across every project. Agents (and you) should never have to guess.

```json
{
  "scripts": {
    "dev": "...",
    "build": "...",
    "start": "...",
    "test": "vitest run",
    "test:watch": "vitest",
    "lint": "eslint .",
    "typecheck": "tsc --noEmit",
    "precommit": "..."
  }
}
```

Monorepo root runs via turbo (`turbo run lint`), package-level scripts do the actual work.

## Next.js: co-locate `route.ts` + `hook.ts`

For every API route, put the SWR hook in the same directory as the route handler. Types defined once in `route.ts`, imported by `hook.ts`.

```
src/app/(protected)/api/interviews/[interviewId]/
├── route.ts         # server handlers + exported response type
└── hook.ts          # "use client" SWR hook, imports types from ./route
```

Why: one grep finds both halves of an endpoint, and TypeScript breaks the build if the hook and handler disagree on shape.

See [`references/nextjs-route-hook.md`](references/nextjs-route-hook.md) for the full pattern (response shape, mutations, conditional fetching, anti-patterns).

## Key Patterns

- **Prisma + Next.js is the default stack**. Deviations should be deliberate.
- **Path aliases**: `@/*` → `./src/*`. Use for cross-cutting imports; relative paths for nearby siblings.
- **Generated code** (`src/generated/**`, OpenAPI clients, route tables) is always gitignored OR marked read-only in comments + excluded from lint. Never edit by hand.
- **Async/await everywhere** — no `.then()` chains in new code.
- **`import type`** for type-only imports (enforced by `verbatimModuleSyntax`).
- **`pnpm.overrides`** in root `package.json` to patch transitive deps (security patches, outdated subdeps).
- **No barrel files (`index.ts` re-exports)** unless you're publishing a library. They hurt tree-shaking and make grep harder.

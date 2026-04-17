# pnpm + turbo monorepos

When to reach for a monorepo, how to wire pnpm workspaces + turbo, and the task topology patterns worth stealing.

## When to use a monorepo

**Yes:**
- Multiple deployable apps that share code (web + bot + pipeline)
- Library + its examples / docs / playground
- Frontend + backend that must share types (tRPC, generated OpenAPI client, Prisma types)

**No:**
- A single app and its tests — just use one package
- "I might need more packages later" — split when the pain is real, not speculative
- Sharing across unrelated projects — that's what npm is for

## Layout

```
repo-root/
├── package.json             # root deps + scripts
├── pnpm-workspace.yaml      # workspace globs
├── turbo.json               # task topology
├── tsconfig.base.json       # shared compilerOptions
├── apps/
│   ├── web/                 # Next.js app
│   └── cli/                 # CLI tool
├── packages/
│   ├── shared/              # shared types / utils
│   └── ui/                  # component library
└── services/
    └── worker/              # background worker
```

`pnpm-workspace.yaml`:
```yaml
packages:
  - apps/*
  - packages/*
  - services/*
```

Root `package.json`:
```json
{
  "name": "my-monorepo",
  "private": true,
  "packageManager": "pnpm@9.15.0",
  "engines": { "node": ">=22" },
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "lint": "turbo run lint",
    "test": "turbo run test",
    "typecheck": "turbo run typecheck"
  },
  "devDependencies": {
    "turbo": "^2.3.0",
    "typescript": "^5.7.0"
  }
}
```

## turbo.json baseline

```jsonc
{
  "$schema": "https://turborepo.com/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "globalEnv": ["NODE_ENV"],
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "build/**"]
    },
    "test": {
      "dependsOn": ["^build"],
      "inputs": ["src/**", "tests/**", "vitest.config.ts"],
      "outputs": []
    },
    "lint": {
      "outputs": []
    },
    "typecheck": {
      "dependsOn": ["^build"],
      "outputs": []
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "clean": { "cache": false }
  }
}
```

### Task topology rules

- **`dependsOn: ["^build"]`** means "build all upstream packages first." Use on `build`, `test`, `typecheck`, `start`.
- **`outputs: []`** on checks that don't produce artifacts (lint, typecheck). Turbo still caches pass/fail.
- **`outputs: ["dist/**", ".next/**"]`** on `build`. Miss these and turbo will re-run every time.
- **`cache: false` + `persistent: true`** on `dev`. Persistent = runs forever, not an artifact producer.
- **`cache: false`** on anything touching secrets, generating code that shouldn't be cached (env pulls, db migrations).
- **`inputs`** on `test` restricts cache invalidation to the source files that matter — changes to `package.json` won't invalidate test cache if `inputs` is set.

### Cross-task dependencies

If `test` depends on generated Prisma types:

```jsonc
{
  "tasks": {
    "db:generate": {
      "cache": false,
      "outputs": ["src/generated/**"]
    },
    "build": {
      "dependsOn": ["^build", "db:generate"],
      "outputs": ["dist/**", ".next/**"]
    },
    "test": {
      "dependsOn": ["^build", "db:generate"],
      "outputs": []
    }
  }
}
```

### Package-specific task overrides

Use `<task>#<package>` to override a single package's task:
```jsonc
{
  "tasks": {
    "build#@scope/dagster": {
      "dependsOn": ["^build"],
      "cache": false,
      "outputs": []
    }
  }
}
```

## Running things

```bash
# Run task across all packages (respects topology)
pnpm turbo run build

# Scope to one package and its deps
pnpm turbo run build --filter=@scope/web

# Scope + include its dependents
pnpm turbo run test --filter=...@scope/shared

# Run in one workspace directly (skip turbo)
pnpm --filter @scope/web dev
```

## Shared tsconfig

Put common compiler options in a root `tsconfig.base.json`:

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "skipLibCheck": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true,
    "esModuleInterop": true
  }
}
```

Each package extends it:

```jsonc
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "dist",
    "declaration": true
  },
  "include": ["src"]
}
```

## Shared deps

Install shared devDeps at the root with `-w`:
```bash
pnpm add -w -D typescript eslint vitest turbo
```

Package-specific runtime deps go in that package:
```bash
pnpm --filter @scope/web add next react react-dom
```

## Common gotchas

- **Peer deps across workspace packages**: `pnpm` will symlink correctly by default, but if a package isn't resolving, check `pnpm-workspace.yaml` globs first, then the consumer's dependency spec (`"@scope/shared": "workspace:*"`).
- **Turbo cache in CI**: set `TURBO_TOKEN` + `TURBO_TEAM` if using remote cache, otherwise cache is local-only per runner.
- **`dev` with `persistent: true`** blocks turbo from running other tasks concurrently unless you mark them all persistent. Use `turbo run dev --parallel` for multi-app dev.

# ESLint (flat config)

The new flat config (`eslint.config.mjs`) is the only supported format going forward. Legacy `.eslintrc.*` is deprecated — don't start new projects with it.

## Install

```bash
pnpm add -D eslint typescript-eslint
```

`typescript-eslint` is the meta-package that pulls in the parser + plugin + a typed config helper.

## Baseline config

For any TypeScript project that isn't Next.js:

```js
// eslint.config.mjs
import tseslint from "typescript-eslint";

export default tseslint.config(
  { ignores: ["dist/**", "node_modules/**", "**/generated/**", "coverage/**"] },
  ...tseslint.configs.recommended,
  {
    rules: {
      "@typescript-eslint/no-unused-vars": [
        "warn",
        { argsIgnorePattern: "^_", varsIgnorePattern: "^_" },
      ],
    },
  },
);
```

## Typed linting (optional, more powerful)

Rules that need type information (like `no-floating-promises`) require the typed preset:

```js
import tseslint from "typescript-eslint";

export default tseslint.config(
  { ignores: ["dist/**", "node_modules/**"] },
  ...tseslint.configs.recommendedTypeChecked,
  {
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      "@typescript-eslint/no-floating-promises": "error",
      "@typescript-eslint/no-misused-promises": "error",
      "@typescript-eslint/no-unused-vars": [
        "warn",
        { argsIgnorePattern: "^_", varsIgnorePattern: "^_" },
      ],
    },
  },
);
```

**Cost:** ~2x slower lint (reads the TS project graph). **Benefit:** catches unawaited promises, misused `await`, and other bugs invisible to untyped lint.

Use typed linting for server/CLI code. Optional for UI.

## Next.js

Next ships its own config. Wire it into flat config via `FlatCompat`:

```js
// eslint.config.mjs
import { FlatCompat } from "@eslint/eslintrc";
import tseslint from "typescript-eslint";

const compat = new FlatCompat({ baseDirectory: import.meta.dirname });

export default tseslint.config(
  { ignores: ["dist/**", ".next/**", "node_modules/**", "src/generated/**"] },
  ...compat.extends("next/core-web-vitals", "next/typescript"),
  {
    rules: {
      "@typescript-eslint/no-unused-vars": [
        "warn",
        { argsIgnorePattern: "^_", varsIgnorePattern: "^_" },
      ],
    },
  },
);
```

Install:
```bash
pnpm add -D eslint-config-next @eslint/eslintrc
```

## Rule picks worth the overhead

| Rule | Why |
|------|-----|
| `@typescript-eslint/no-floating-promises` (typed) | Unawaited promises swallow errors silently |
| `@typescript-eslint/no-misused-promises` (typed) | Catches `onClick={async () => {...}}` on something that expects sync |
| `@typescript-eslint/no-unused-vars` (with `^_` ignore) | Auto-fixable, better than tsconfig's version |
| `no-console` (`["warn", { allow: ["warn", "error"] }]`) | Keeps log noise down in production code |
| `eqeqeq` | `===` only, no coercion bugs |

## Rules to skip

- **`import/order`** — modern editors sort imports automatically. Adding the `eslint-plugin-import` dep for this alone is rarely worth it.
- **`@typescript-eslint/explicit-function-return-type`** — noisy. Inference is the point of TS.
- **`@typescript-eslint/explicit-module-boundary-types`** — same.
- **Prettier plugins / `eslint-config-prettier`** — no formatter in this stack.

## Scripts

```json
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix"
  }
}
```

ESLint 9+ picks up `eslint.config.mjs` automatically. No `--ext` flag needed.

## CI

```yaml
- run: pnpm lint
```

Fails on errors. Warnings don't fail the build — use `--max-warnings 0` if you want zero-tolerance.

## Ignores

The first argument to `tseslint.config` (a bare object with `ignores`) replaces the old `.eslintignore` file. Always ignore:
- `dist/`, `.next/`, `build/`, `coverage/` — generated output
- `node_modules/` — not yours
- `**/generated/**` — Prisma client, OpenAPI clients, auto-generated routes
- Any file that's machine-written

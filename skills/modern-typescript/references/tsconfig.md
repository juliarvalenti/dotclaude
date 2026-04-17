# tsconfig flags, per flag

Why each baseline flag is on, what each skipped flag does, and when to reach for non-defaults.

## On by default

### `strict: true`
Enables a bundle: `noImplicitAny`, `strictNullChecks`, `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`, `alwaysStrict`, `noImplicitThis`, `useUnknownInCatchVariables`.

**Non-negotiable.** Turning this off means most of TypeScript's value is off.

### `noUncheckedIndexedAccess: true`
Makes `arr[0]` and `record[key]` return `T | undefined` instead of `T`. Forces you to check before use.

**Catches real bugs** — lookups that miss at runtime are the most common class of "works on my machine" issues. Cost: a few extra `if` checks. Worth it.

### `noImplicitOverride: true`
Methods that override a parent must use the `override` keyword.

**Low cost, high value.** Catches refactor drift — if you rename the parent method, the child no longer overrides, and TS will tell you.

### `noFallthroughCasesInSwitch: true`
Requires `break` / `return` / `throw` in every `case`, or an explicit `// fallthrough` comment.

**Cheap insurance.** Fallthrough bugs are silent killers.

### `skipLibCheck: true`
Don't type-check `.d.ts` files in `node_modules`.

**Pragmatic.** Lib authors break their own types constantly; you can't fix them. Off-by-default was a mistake.

### `resolveJsonModule: true`
`import pkg from "./package.json"` works and gets typed.

### `isolatedModules: true`
Each file must be independently transpilable (no cross-file const-enum merging, etc.). Required by bundlers like SWC / esbuild / Babel.

### `verbatimModuleSyntax: true`
`import { Foo }` stays as runtime import; `import type { Foo }` is a pure type import. No silent elision.

**Forces discipline** — `import type` everywhere it should be. Makes bundler behavior predictable.

## Off by default (but sometimes considered)

### `exactOptionalPropertyTypes: false`
With this on, `{ x?: number }` and `{ x?: number | undefined }` become different types, and you can't assign `undefined` explicitly to an optional.

**Skip.** It catches almost nothing real and breaks a huge number of library types. The noise-to-signal is bad.

### `noPropertyAccessFromIndexSignature: false`
Forces `obj["key"]` instead of `obj.key` when the type is an index signature.

**Skip.** Pedantic. Fights normal code. Rarely catches a real bug.

### `noUnusedLocals` / `noUnusedParameters`
Flag unused variables / parameters as errors.

**Skip — let ESLint do this instead.** ESLint's `@typescript-eslint/no-unused-vars` is auto-fixable, configurable per-file, and can ignore underscore-prefixed names. TS version is rigid and blocks the build.

### `allowImportingTsExtensions`
Lets you write `import { x } from "./foo.ts"`.

**Skip unless your bundler/runtime requires it** (Deno, some Bun configs). Node + pnpm + tsc don't need it.

## Per-target overrides

### Next.js app
```jsonc
{
  "target": "ES2022",
  "lib": ["ES2022", "DOM", "DOM.Iterable"],
  "module": "ESNext",
  "moduleResolution": "Bundler",
  "jsx": "preserve",
  "noEmit": true,
  "incremental": true,
  "plugins": [{ "name": "next" }],
  "paths": { "@/*": ["./src/*"] }
}
```
`"moduleResolution": "Bundler"` is correct for Next — lets you import without extensions and matches webpack/turbopack resolution.

### Library (published to npm)
```jsonc
{
  "module": "NodeNext",
  "moduleResolution": "NodeNext",
  "declaration": true,
  "declarationMap": true,
  "sourceMap": true,
  "outDir": "dist"
}
```
Node16/NodeNext enforces file extensions in imports — which is what real ESM on Node requires.

### CLI tool
Same as library but `"declaration": false` (no one imports your CLI as a library).

### Monorepo package with project references
Add `"composite": true` to each package, then wire up `references` in consumers. Lets `tsc --build` incrementally compile only what changed.

## On the "ESLint vs tsconfig" question

**Some checks are only possible in the type checker** because ESLint doesn't have full type info even with the typed plugin:
- Exhaustive null checks (`strictNullChecks`)
- Array access safety (`noUncheckedIndexedAccess`)
- Override correctness (`noImplicitOverride`)

**Some checks are best in ESLint** because they're lint-shaped, not type-shaped:
- Unused variables (auto-fixable, per-file overridable)
- Import ordering (not a type concern)
- Naming conventions
- "No console.log in production" style rules

**Rule of thumb:** if the check is about type relationships, use tsconfig. If it's about code style or usage patterns, use ESLint. There's no reason to pick one over the other — use both.

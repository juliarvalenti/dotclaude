# Module systems: ESM vs CJS

The decision that causes more wasted hours than it should. Here's the short version.

## Defaults by project type

| Project | Default | `"type"` | tsconfig `module` |
|---------|---------|----------|-------------------|
| Next.js app | Bundler decides | omit | `ESNext` / `Preserve` |
| Node library (published to npm) | ESM | `"module"` | `NodeNext` |
| CLI tool | ESM | `"module"` | `NodeNext` |
| Node service | ESM | `"module"` | `NodeNext` |
| Electron main process | CJS (until forced otherwise) | omit or `"commonjs"` | `CommonJS` |
| Legacy AWS Lambda / serverless | check runtime first | depends | depends |

**Greenfield rule:** start ESM. Go back to CJS only when a required dep blocks you.

## Decision tree

1. **Is it a Next.js app?** The bundler handles module resolution. Use `"moduleResolution": "Bundler"`, don't set `"type"`.
2. **Does it need to run directly on Node (CLI, service, lambda)?** ESM (`"type": "module"`, `"module": "NodeNext"`, `"moduleResolution": "NodeNext"`).
3. **Is it a library you're publishing?** Default to ESM. Provide a CJS export only if you genuinely have CJS consumers asking.
4. **Are any of your critical deps CJS-only?** Most work under ESM now. If one truly doesn't (and `createRequire` won't do), you're stuck with CJS.

## The three knobs that must agree

- **`package.json` `"type"`** — tells Node how to interpret `.js` files. `"module"` = ESM, `"commonjs"` (or unset) = CJS.
- **`tsconfig.json` `"module"`** — what module syntax TS emits.
- **`tsconfig.json` `"moduleResolution"`** — how TS resolves imports.

**Matching pairs:**

| Scenario | `"type"` | `"module"` | `"moduleResolution"` |
|----------|----------|------------|----------------------|
| ESM on Node | `"module"` | `"NodeNext"` | `"NodeNext"` |
| CJS on Node | `"commonjs"` (or omit) | `"CommonJS"` | `"Node10"` (legacy) |
| Bundler (Next / Vite / webpack) | omit | `"ESNext"` or `"Preserve"` | `"Bundler"` |

Mismatching these is 80% of "my imports aren't resolving" bugs.

## Reading a package's `exports` field

Modern packages use `exports` in their `package.json` to declare what can be imported. Example:

```json
{
  "type": "module",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./client": {
      "types": "./dist/client.d.ts",
      "import": "./dist/client.js"
    }
  }
}
```

- **`types`** must come first in the conditions object (TS reads top-to-bottom and the first match wins).
- **`import`** resolves in ESM consumers, **`require`** in CJS.
- **`./client`** is a subpath export. Consumers can do `import x from "pkg/client"`; anything not declared is blocked.
- **No `exports` field** → consumers can import any file in the package. Fragile but works.

When publishing a library, declare every public entry in `exports`. It's your API contract.

## Writing ESM on Node

- **File extensions in imports are required**: `import { foo } from "./foo.js"` — yes, `.js` even when the file is `foo.ts`. TypeScript with `moduleResolution: NodeNext` requires this.
- **No `__dirname` / `__filename`** — use `import.meta.url`:
  ```ts
  import { fileURLToPath } from "node:url";
  import { dirname } from "node:path";
  const __dirname = dirname(fileURLToPath(import.meta.url));
  ```
- **Dynamic import of CJS**: works. `const pkg = await import("some-cjs-pkg");` — the default export is under `pkg.default` or `pkg` depending on the package.
- **`require` is gone** unless you do `import { createRequire } from "node:module"; const require = createRequire(import.meta.url);`.
- **Top-level `await` works** in ESM modules. Doesn't work in CJS.

## Writing CJS on Node

Standard stuff. `module.exports`, `require`. Nothing new to learn — this is the old world.

If you need to import an ESM-only package from CJS, you can't use `require` — must use dynamic `import()`:

```ts
async function loadEsmOnly() {
  const { foo } = await import("esm-only-pkg");
  return foo;
}
```

## Dual-package publish (if you really have to)

Only do this if you have known CJS consumers. Otherwise ESM-only is simpler.

```json
{
  "type": "module",
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    }
  }
}
```

Build with `tsup --format esm,cjs --dts` or similar. Don't hand-roll two builds.

**Hazard:** "dual package hazard" — consumers end up with two copies of your singleton state (one per module system). Avoid if your package holds state.

## Known CJS-only packages to watch for

Most popular packages are dual-export or ESM-only now. Occasionally you'll still hit older ones — when you do, dynamic `import()` from CJS is the escape hatch.

Check a package: `npm view <pkg> exports` or open its `package.json` in `node_modules`.

## TL;DR

- New project → ESM.
- Tooling triangle (`type` + `module` + `moduleResolution`) must match.
- `.js` extensions in imports. Yes, even for `.ts` files.
- Bundler projects (Next/Vite) are different: `"moduleResolution": "Bundler"` and don't set `"type"`.
- Dual-publish only when you have a real CJS consumer.

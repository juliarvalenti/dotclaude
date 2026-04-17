# Validation: Prisma first, Zod at boundaries

Type information has a cost. Every schema you maintain is a thing that can drift. Reach for the lightest option that catches the real errors.

## Hierarchy (most preferred → least)

1. **Prisma-generated types** — for anything that exists in the database.
2. **Prisma-derived types** — query-shape types built from Prisma generics.
3. **TypeScript-inferred types** — from actual runtime code (`typeof config`, `ReturnType<typeof fn>`).
4. **Zod schemas** — only for untrusted boundaries.

## 1. Prisma types directly

```ts
import type { User, Organization, Session } from "@/generated/prisma/client";

function greet(user: User) { ... }
```

**Never redefine a database entity type.** If `User` has 17 fields in Prisma, don't write `interface User { id: string; name: string; }` somewhere else — it will drift, and someone will consume the wrong one.

## 2. Query-shape types

When a handler returns a subset or extension of a model:

```ts
import type { Prisma } from "@/generated/prisma/client";

// Full shape with a relation included
type UserWithOrg = Prisma.UserGetPayload<{ include: { organization: true } }>;

// Partial shape from select
type UserListItem = Prisma.UserGetPayload<{ select: { id: true; name: true; email: true } }>;
```

Export these from `src/types/` (or co-located with the query) so hooks and components can import them.

## 3. Inferred types

```ts
// Config derived from runtime object
export const config = { host: "localhost", port: 3000 } as const;
export type Config = typeof config;

// Return type inferred from the function
export type SessionSummary = ReturnType<typeof getSessionSummary>;
```

When the source of truth is code, not a schema, infer the type from the code.

## 4. Zod — only at boundaries

**Use Zod for:**
- Form input (React Hook Form + `zodResolver`, or manual `.parse()` on submit)
- API route inputs (`req.json()` → `schema.parse(...)`)
- Webhook payloads
- Environment variables (`process.env` → parsed schema)
- Parsed config files (YAML / TOML / JSON)
- Any untyped payload crossing the runtime boundary

**Don't use Zod for:**
- Internal function args between your own modules (TypeScript already has types)
- Database entity shape (Prisma)
- Return shapes from your own API (TypeScript types + the route.ts + hook.ts pattern)

## Where schemas live

Centralize per boundary:

```
src/
├── schemas/
│   ├── env.ts                 # process.env validation
│   ├── forms/
│   │   ├── signup.ts
│   │   └── support-request.ts
│   └── webhooks/
│       └── stripe.ts
└── types/
    └── api/
        └── json-fields.ts     # Prisma JSON column shapes
```

One schema per input shape, exported + reused:

```ts
// src/schemas/forms/support-request.ts
import { z } from "zod";

export const supportRequestSchema = z.object({
  email: z.email(),
  subject: z.string().min(1).max(200),
  body: z.string().min(1).max(10_000),
});

export type SupportRequest = z.infer<typeof supportRequestSchema>;
```

## `parse` vs `safeParse`

**Default to `.parse()`** + try/catch at the route boundary. Don't plumb `Result` types through application code.

```ts
// route.ts
export async function POST(req: Request) {
  try {
    const input = supportRequestSchema.parse(await req.json());
    await createSupportRequest(input);
    return Response.json({ ok: true });
  } catch (err) {
    if (err instanceof z.ZodError) {
      return Response.json({ error: err.message }, { status: 400 });
    }
    throw err;
  }
}
```

**Use `.safeParse()`** only when validation failure is part of normal control flow (e.g., trying multiple schemas, returning structured errors to a form for inline display):

```ts
// inline form errors
const result = supportRequestSchema.safeParse(formData);
if (!result.success) {
  setFormErrors(result.error.flatten().fieldErrors);
  return;
}
await submit(result.data);
```

## Prisma JSON columns

Prisma's `Json` columns are typed as `JsonValue` — useless for real work. Define a Zod schema per JSON column and use it to parse on read:

```ts
// src/types/api/json-fields.ts
export const userPreferencesSchema = z.object({
  theme: z.enum(["light", "dark", "auto"]).default("auto"),
  notifications: z.object({ email: z.boolean(), push: z.boolean() }),
});

export type UserPreferences = z.infer<typeof userPreferencesSchema>;

// when reading
const prefs = userPreferencesSchema.parse(user.preferences);
```

When writing, use the inferred type on the input so TS catches mismatches before they hit the DB.

## Env variables

```ts
// src/schemas/env.ts
import { z } from "zod";

const envSchema = z.object({
  DATABASE_URL: z.url(),
  NEXT_PUBLIC_API_URL: z.url(),
  NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
});

export const env = envSchema.parse(process.env);
```

Import `env` everywhere instead of `process.env.FOO`. Missing / malformed env now fails at boot with a clear error, not at the first request.

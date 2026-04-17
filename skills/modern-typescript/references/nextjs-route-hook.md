# Next.js: co-locate `route.ts` + `hook.ts`

For every API route in a Next.js app, put the SWR hook in the same directory as the route handler. Types defined once in `route.ts`, imported by `hook.ts`.

## Why

**Server and client drift if they live far apart.** A handler returns `{ success, interview }`, the hook expects `{ data: { interview } }`, nobody notices until runtime. Co-locating means:

- One grep finds both halves of an endpoint
- Response types are defined in the route and imported by the hook — TypeScript breaks the build if they disagree
- Renaming or changing the endpoint touches two files in the same folder, not two files across the tree
- New contributors find the pattern by looking at any existing route

## Layout

```
src/app/(protected)/api/interviews/[interviewId]/
├── route.ts         # server: GET / POST / PATCH / DELETE handlers
└── hook.ts          # client: useInterview() SWR hook
```

One folder = one endpoint. If a route has sub-paths, each sub-path gets its own folder with its own `route.ts` + `hook.ts`.

## route.ts — export handler + response type

```ts
import { NextResponse } from "next/server";
import { withDynamicAuth } from "@/lib/auth-middleware";
import type { TransformedInterview } from "@/lib/api/transformers";

export interface InterviewApiResponse {
  success: boolean;
  interview: TransformedInterview;
}

export const GET = withDynamicAuth(async (_req, _orgCtx, context) => {
  try {
    const { interviewId } = await context.params;
    if (!interviewId) {
      return NextResponse.json({ error: "Interview ID is required" }, { status: 400 });
    }
    const interview = await fetchInterview(interviewId);
    if (!interview) {
      return NextResponse.json({ error: "Interview not found" }, { status: 404 });
    }
    return NextResponse.json({ success: true, interview } satisfies InterviewApiResponse);
  } catch (err) {
    logApiError("Get interview error:", err);
    return NextResponse.json({ error: "Failed to get interview" }, { status: 500 });
  }
});
```

**Rules:**
- Response type is `export`ed, named `<Resource>ApiResponse`.
- Use `satisfies InterviewApiResponse` on the response body so TypeScript verifies the shape without widening the literal.
- Handler is wrapped in an auth middleware (`withStaticAuth`, `withDynamicTeacherAuth`, etc.) — never raw.
- Dynamic params: always `await context.params` before accessing.
- Errors: 400 for missing input, 404 for not found, 403 for forbidden, 500 with a user-friendly message for unexpected.

## hook.ts — import types from `./route`

```ts
"use client";

import useSWR from "swr";
import { createTypedSwrFetcher } from "@/lib/utils";
import { dynamicRoutes } from "@/lib/routes";
import type { InterviewApiResponse } from "./route";
import type { TransformedInterview } from "@/lib/api/transformers";

interface UseInterviewOptions {
  autoRefresh?: boolean;
  refreshInterval?: number;
}

interface UseInterviewReturn {
  interview: TransformedInterview | undefined;
  isLoading: boolean;
  error: Error | undefined;
  refresh: () => Promise<void>;
}

const fetcher = createTypedSwrFetcher<InterviewApiResponse>();

export const useInterview = (
  interviewId: string | null,
  options: UseInterviewOptions = {},
): UseInterviewReturn => {
  const { autoRefresh = false, refreshInterval = 30_000 } = options;

  const { data, error, isLoading, mutate } = useSWR<InterviewApiResponse>(
    interviewId ? dynamicRoutes.api.interviews.byId(interviewId) : null,
    fetcher,
    {
      refreshInterval: autoRefresh ? refreshInterval : 0,
      revalidateOnFocus: true,
      revalidateOnReconnect: true,
    },
  );

  return {
    interview: data?.interview,
    isLoading,
    error,
    refresh: async () => { await mutate(); },
  };
};
```

**Rules:**
- `"use client"` at the top — always.
- Import the response type from `./route` — never redeclare it. If the route changes shape, the hook breaks the build.
- SWR key is `null` when required params are missing — SWR skips the fetch cleanly.
- Use `dynamicRoutes.*` / `urls.*` from the generated routes file — never hardcode path strings.
- Return shape is consistent: `{ <resource>, isLoading, error, refresh }`. If the route returns a list, use the plural (`interviews`).
- `refresh` wraps `mutate()` so consumers don't depend on the SWR API.
- Options go in a second optional `options` arg with sensible defaults.

## Variants

**List endpoint:**
```ts
// hook.ts
export const useInterviews = (): UseInterviewsReturn => {
  const { data, error, isLoading, mutate } = useSWR<InterviewListApiResponse>(
    urls.api.interviews.root,
    fetcher,
  );
  return { interviews: data?.interviews, isLoading, error, refresh: async () => { await mutate(); } };
};
```

**Mutations (POST / PATCH / DELETE):** use SWR's `useSWRMutation`, or a plain async function. Still co-locate in the same `hook.ts`:

```ts
export const useUpdateInterview = () => {
  return async (id: string, patch: Partial<InterviewPatch>) => {
    const res = await fetch(dynamicRoutes.api.interviews.byId(id), {
      method: "PATCH",
      headers: { "content-type": "application/json" },
      body: JSON.stringify(patch),
    });
    if (!res.ok) throw new Error(await res.text());
    return res.json() as Promise<InterviewApiResponse>;
  };
};
```

**Conditional fetching with dependencies:** chain SWR keys — pass `null` when a dep hasn't loaded, SWR waits.

```ts
const { data: user } = useUser();
const { data: prefs } = useSWR(user ? dynamicRoutes.api.users.prefs(user.id) : null, fetcher);
```

## Anti-patterns

- ❌ **Fetching in `useEffect`** — never. Always SWR (or `useSWRMutation`).
- ❌ **Hardcoding route strings** — `"/api/interviews/" + id`. Use `dynamicRoutes`.
- ❌ **Redeclaring the response type in the hook** — import from `./route`.
- ❌ **Putting the hook in `@/hooks/use-interview.ts`** while the route is 8 directories away. Co-locate.
- ❌ **Raw handler without an auth wrapper** — always use `withStaticAuth` / `withDynamicAuth` etc.
- ❌ **Forgetting `await context.params`** — dynamic routes in Next 15+ require the await.

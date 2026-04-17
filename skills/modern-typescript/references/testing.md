# Vitest

Default test runner. Fast, TS-native, Vite-based, API-compatible with Jest for most cases.

## Install

```bash
pnpm add -D vitest
```

For DOM testing:
```bash
pnpm add -D vitest @vitest/ui jsdom @testing-library/react @testing-library/jest-dom
```

## Config

Minimal `vitest.config.ts`:

```ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    include: ["src/**/*.test.ts", "tests/**/*.test.ts"],
    environment: "node",
  },
});
```

With path aliases (mirror your tsconfig):

```ts
import { defineConfig } from "vitest/config";
import { fileURLToPath } from "node:url";

export default defineConfig({
  test: {
    include: ["src/**/*.test.{ts,tsx}"],
    environment: "jsdom",              // for React / DOM code
    setupFiles: ["./vitest.setup.ts"], // jest-dom matchers, etc.
    globals: true,                      // enables describe/it/expect without import
  },
  resolve: {
    alias: {
      "@": fileURLToPath(new URL("./src", import.meta.url)),
    },
  },
});
```

`vitest.setup.ts` (if using jest-dom):
```ts
import "@testing-library/jest-dom/vitest";
```

## Scripts

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage"
  }
}
```

CI runs `pnpm test`. Local dev uses `test:watch`.

## File conventions

- **Unit tests**: co-locate with source — `src/utils/cn.ts` → `src/utils/cn.test.ts`.
- **Integration tests**: under `tests/integration/` at the project root. Slower, may hit a real DB.
- **E2E tests**: separate tool (Playwright). Not Vitest.

## Patterns

### Plain unit test

```ts
import { describe, it, expect } from "vitest";
import { slugify } from "./slugify";

describe("slugify", () => {
  it("lowercases and hyphenates", () => {
    expect(slugify("Hello World")).toBe("hello-world");
  });

  it("strips punctuation", () => {
    expect(slugify("Hello, World!")).toBe("hello-world");
  });
});
```

With `globals: true` in the config, you can skip the imports.

### Mocking modules

```ts
import { vi } from "vitest";

vi.mock("@/lib/email", () => ({
  sendEmail: vi.fn().mockResolvedValue({ id: "msg_123" }),
}));

// In the test
import { sendEmail } from "@/lib/email";
expect(sendEmail).toHaveBeenCalledWith({ to: "a@b.com", subject: "hi" });
```

### Mocking with partial real impl

```ts
vi.mock("@/lib/db", async () => {
  const actual = await vi.importActual<typeof import("@/lib/db")>("@/lib/db");
  return { ...actual, expensiveQuery: vi.fn().mockResolvedValue([]) };
});
```

### Setup / teardown

```ts
import { beforeEach, afterAll } from "vitest";

beforeEach(() => {
  vi.useFakeTimers();
});

afterAll(() => {
  vi.useRealTimers();
});
```

### Fixtures (parameterized tests)

```ts
it.each([
  ["Hello World", "hello-world"],
  ["Foo Bar Baz", "foo-bar-baz"],
])("slugify(%s) → %s", (input, expected) => {
  expect(slugify(input)).toBe(expected);
});
```

## Testing Next.js API routes

Import the handler directly. Build a `Request`, call, assert on `Response`.

```ts
import { GET } from "@/app/api/interviews/[interviewId]/route";
import { describe, it, expect, vi } from "vitest";

vi.mock("@/lib/context", () => ({
  getOrganizationDB: () => ({
    interview: {
      findUnique: vi.fn().mockResolvedValue({ id: "i_1", name: "Test" }),
    },
  }),
}));

describe("GET /api/interviews/[interviewId]", () => {
  it("returns the interview", async () => {
    const req = new Request("http://localhost/api/interviews/i_1");
    const res = await GET(req, {}, { params: Promise.resolve({ interviewId: "i_1" }) });
    expect(res.status).toBe(200);
    const body = await res.json();
    expect(body.success).toBe(true);
    expect(body.interview.id).toBe("i_1");
  });
});
```

**Caveats:**
- Auth middleware wraps the handler — either test with auth mocked, or export an unwrapped variant for tests.
- `context.params` is a Promise in Next 15+ — wrap with `Promise.resolve(...)`.

## Testing React components

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, it, expect, vi } from "vitest";
import { Button } from "./button";

describe("<Button>", () => {
  it("fires onClick when clicked", async () => {
    const onClick = vi.fn();
    render(<Button onClick={onClick}>Go</Button>);
    await userEvent.click(screen.getByRole("button", { name: /go/i }));
    expect(onClick).toHaveBeenCalledOnce();
  });
});
```

Install `@testing-library/user-event` for realistic interactions (typing, focus, etc.).

## Testing SWR hooks

Wrap in `SWRConfig` with a clean cache per test:

```tsx
import { SWRConfig } from "swr";
import { renderHook, waitFor } from "@testing-library/react";

const wrapper = ({ children }: { children: React.ReactNode }) => (
  <SWRConfig value={{ provider: () => new Map(), dedupingInterval: 0 }}>
    {children}
  </SWRConfig>
);

it("useInterview fetches the interview", async () => {
  const { result } = renderHook(() => useInterview("i_1"), { wrapper });
  await waitFor(() => expect(result.current.isLoading).toBe(false));
  expect(result.current.interview?.id).toBe("i_1");
});
```

Mock `fetch` or the fetcher at the module level.

## Integration tests hitting a real DB

- Run against a throwaway database (docker, sqlite file, ephemeral Postgres container).
- Wrap each test in a transaction and roll back — fastest way to isolate tests.
- Guard with an env var (`VITEST_INTEGRATION=1`) so they don't run by default: `test: { env: { ... } }` or a separate config.

```ts
// vitest.integration.config.ts
export default defineConfig({
  test: {
    include: ["tests/integration/**/*.test.ts"],
    environment: "node",
    testTimeout: 30_000,
  },
});
```

Script: `"test:integration": "vitest run --config vitest.integration.config.ts"`.

## Coverage

```bash
pnpm add -D @vitest/coverage-v8
```

```ts
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: "v8",
      reporter: ["text", "html"],
      exclude: ["**/*.config.*", "**/generated/**", "dist/**"],
    },
  },
});
```

Don't enforce coverage thresholds reflexively — they incentivize writing bad tests. Use coverage as a discovery tool for untested code paths.

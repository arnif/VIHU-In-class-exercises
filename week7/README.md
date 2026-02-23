# Week 7: E2E Testing with Playwright & Integration Testing with MSW

## Objective

Learn two essential testing strategies for web applications: **End-to-End (E2E) testing** with Playwright and **Integration testing** with Mock Service Worker (MSW). By the end, you'll understand when to use each approach and be able to write both types of tests for a Next.js application.

---

## Why Two Types of Tests?

Unit tests (Week 3) test individual functions in isolation. But real applications have a UI that talks to APIs, and you need to test that everything works together. There are two complementary approaches:

| | E2E Tests (Playwright) | Integration Tests (MSW + Vitest) |
|---|---|---|
| **What it tests** | Full app in a real browser | UI components with mocked API responses |
| **Browser** | Real Chromium/Firefox/WebKit | No browser needed |
| **API calls** | Hit the real server | Intercepted and mocked by MSW |
| **Speed** | Slower (seconds per test) | Fast (milliseconds per test) |
| **How many?** | Few — critical happy paths only | Many — cover edge cases and error states |
| **Catches** | Real-world integration issues | UI logic bugs, loading/error states |
| **Runs in CI** | Yes | Yes |

### The Testing Strategy

Think of it like checking a car:

- **E2E tests** = Taking the car for a test drive. You start it, drive it, brake, park. Few tests, but they prove the whole thing works.
- **Integration tests** = Testing each system (engine, brakes, lights) on a bench with simulated inputs. Many tests, fast feedback, easy to test failure scenarios.

```
┌─────────────────────────────────────────────┐
│              E2E Tests (Few)                 │
│         Real browser, real server            │
│       "Does the whole app work?"             │
├─────────────────────────────────────────────┤
│         Integration Tests (Many)            │
│       Real UI components, mocked APIs        │
│   "Does the UI handle all scenarios?"        │
├─────────────────────────────────────────────┤
│           Unit Tests (Many)                 │
│        Pure functions, no UI/API             │
│     "Does this function work?"               │
└─────────────────────────────────────────────┘
```

---

## Prerequisites

- **Bun** installed (https://bun.sh/)
- **Node.js** installed (Playwright requires Node.js)
- Basic knowledge of TypeScript and React

---

## Project Structure (What You'll Build)

```
week-7-testing/
├── .github/
│   └── workflows/
│       └── tests.yml
├── src/
│   └── app/
│       ├── layout.tsx
│       ├── page.tsx
│       ├── api/
│       │   └── users/
│       │       └── route.ts
│       └── components/
│           └── UserList.tsx
├── e2e/
│   └── home.spec.ts
├── integration/
│   ├── setup.ts
│   └── UserList.test.tsx
├── mocks/
│   ├── handlers.ts
│   └── server.ts
├── playwright.config.ts
├── vitest.config.ts
├── package.json
└── tsconfig.json
```

---

## Part 1: Set Up the Application

We need a small Next.js app to test. It will display a list of users fetched from an API.

### Exercise 1: Create a Next.js Project

1. Create the project:
   ```bash
   bunx create-next-app@latest week-7-testing --typescript --app --tailwind --eslint --src-dir --import-alias "@/*" --use-bun
   ```
   You'll be asked "Would you like to use React Compiler?" — select **No** (the default). Accept the defaults for any other prompts.

2. Navigate into the project:
   ```bash
   cd week-7-testing
   ```

3. Verify it runs:
   ```bash
   bun dev
   ```
   Open http://localhost:3000 and confirm you see the Next.js default page. Stop the dev server (`Ctrl+C`).

### Exercise 2: Create a User List Component

This component fetches and displays users from an API. We'll test this with both approaches.

1. Create `src/app/components/UserList.tsx`:

   ```tsx
   "use client";

   import { useEffect, useState } from "react";

   interface User {
     id: number;
     name: string;
     email: string;
   }

   export default function UserList() {
     const [users, setUsers] = useState<User[]>([]);
     const [loading, setLoading] = useState(true);
     const [error, setError] = useState<string | null>(null);

     useEffect(() => {
       fetch("/api/users")
         .then((res) => {
           if (!res.ok) throw new Error("Failed to fetch users");
           return res.json();
         })
         .then((data) => {
           setUsers(data);
           setLoading(false);
         })
         .catch((err) => {
           setError(err.message);
           setLoading(false);
         });
     }, []);

     if (loading) return <p data-testid="loading">Loading users...</p>;
     if (error) return <p data-testid="error">Error: {error}</p>;

     return (
       <div data-testid="user-list">
         <h2>Users</h2>
         <ul>
           {users.map((user) => (
             <li key={user.id} data-testid="user-item">
               <strong>{user.name}</strong> — {user.email}
             </li>
           ))}
         </ul>
       </div>
     );
   }
   ```

   **Why `data-testid`?** Test IDs give your tests a stable way to find elements. Unlike CSS classes or text content, they won't change when you redesign the UI.

2. Create a simple API route at `src/app/api/users/route.ts`:

   ```typescript
   import { NextResponse } from "next/server";

   const users = [
     { id: 1, name: "Alice Johnson", email: "alice@example.com" },
     { id: 2, name: "Bob Smith", email: "bob@example.com" },
     { id: 3, name: "Charlie Brown", email: "charlie@example.com" },
   ];

   export async function GET() {
     return NextResponse.json(users);
   }
   ```

3. Update `src/app/page.tsx` to use the component:

   ```tsx
   import UserList from "./components/UserList";

   export default function Home() {
     return (
       <main style={{ padding: "2rem" }}>
         <h1>Week 7: Testing Exercise</h1>
         <UserList />
       </main>
     );
   }
   ```

4. Verify it works:
   ```bash
   bun dev
   ```
   Open http://localhost:3000. You should see the heading and a list of three users. Stop the dev server.

---

## Part 2: E2E Testing with Playwright

E2E tests launch a real browser, navigate to your running application, and interact with it the same way a user would. These tests verify that the entire stack works — from the UI to the API to the database.

### Exercise 3: Install and Configure Playwright

1. Install Playwright:
   ```bash
   bun add -d @playwright/test
   ```

2. Install the browsers Playwright needs:
   ```bash
   bunx playwright install
   ```
   This downloads Chromium, Firefox, and WebKit. It may take a minute.

3. Create `playwright.config.ts` in the project root:

   ```typescript
   import { defineConfig } from "@playwright/test";

   export default defineConfig({
     testDir: "./e2e",
     timeout: 30000,
     retries: 0,
     use: {
       baseURL: "http://localhost:3000",
       trace: "on-first-retry",
     },
     webServer: {
       command: "bun dev",
       port: 3000,
       reuseExistingServer: !process.env.CI,
     },
   });
   ```

   **What this does:**
   - `testDir` — Tells Playwright where to find test files
   - `webServer` — Automatically starts your Next.js dev server before running tests
   - `baseURL` — All `page.goto("/")` calls resolve against this URL
   - `reuseExistingServer` — In development, reuses your running dev server; in CI, starts a fresh one

4. Add a script to `package.json`:
   ```json
   {
     "scripts": {
       "test:e2e": "bunx playwright test"
     }
   }
   ```

### Exercise 4: Write Your First E2E Test

1. Create the test directory and file:
   ```bash
   mkdir e2e
   ```

2. Create `e2e/home.spec.ts`:

   ```typescript
   import { test, expect } from "@playwright/test";

   test.describe("Home Page", () => {
     test("should display the page heading", async ({ page }) => {
       await page.goto("/");

       await expect(
         page.getByRole("heading", { name: "Week 7: Testing Exercise" })
       ).toBeVisible();
     });

     test("should load and display users", async ({ page }) => {
       await page.goto("/");

       // Wait for loading to finish
       await expect(page.getByTestId("loading")).toBeVisible();
       await expect(page.getByTestId("user-list")).toBeVisible();

       // Verify users are displayed
       const userItems = page.getByTestId("user-item");
       await expect(userItems).toHaveCount(3);

       // Check specific user content
       await expect(userItems.first()).toContainText("Alice Johnson");
     });
   });
   ```

   **Key Playwright concepts:**
   - `page` — A real browser tab you can control
   - `page.goto("/")` — Navigate to a URL
   - `page.getByRole()` — Find elements by their accessible role (preferred method)
   - `page.getByTestId()` — Find elements by `data-testid` attribute
   - `expect(...).toBeVisible()` — Assert that an element is visible on the page
   - Playwright auto-waits — It automatically retries assertions until they pass or timeout

3. Run the E2E tests:
   ```bash
   bun run test:e2e
   ```

   Playwright will start the Next.js dev server, launch a browser, run the tests, and report results.

### Exercise 5: Test User Interaction

Let's add a search feature and test it end-to-end.

1. Update `src/app/components/UserList.tsx` to add search:

   ```tsx
   "use client";

   import { useEffect, useState } from "react";

   interface User {
     id: number;
     name: string;
     email: string;
   }

   export default function UserList() {
     const [users, setUsers] = useState<User[]>([]);
     const [loading, setLoading] = useState(true);
     const [error, setError] = useState<string | null>(null);
     const [search, setSearch] = useState("");

     useEffect(() => {
       fetch("/api/users")
         .then((res) => {
           if (!res.ok) throw new Error("Failed to fetch users");
           return res.json();
         })
         .then((data) => {
           setUsers(data);
           setLoading(false);
         })
         .catch((err) => {
           setError(err.message);
           setLoading(false);
         });
     }, []);

     if (loading) return <p data-testid="loading">Loading users...</p>;
     if (error) return <p data-testid="error">Error: {error}</p>;

     const filteredUsers = users.filter(
       (user) =>
         user.name.toLowerCase().includes(search.toLowerCase()) ||
         user.email.toLowerCase().includes(search.toLowerCase())
     );

     return (
       <div data-testid="user-list">
         <h2>Users</h2>
         <input
           type="text"
           placeholder="Search users..."
           value={search}
           onChange={(e) => setSearch(e.target.value)}
           data-testid="search-input"
         />
         <ul>
           {filteredUsers.map((user) => (
             <li key={user.id} data-testid="user-item">
               <strong>{user.name}</strong> — {user.email}
             </li>
           ))}
         </ul>
         {filteredUsers.length === 0 && (
           <p data-testid="no-results">No users found</p>
         )}
       </div>
     );
   }
   ```

2. Add a search test to `e2e/home.spec.ts`:

   ```typescript
   test("should filter users when searching", async ({ page }) => {
     await page.goto("/");

     // Wait for users to load
     await expect(page.getByTestId("user-list")).toBeVisible();

     // Type in the search box
     await page.getByTestId("search-input").fill("Alice");

     // Should show only Alice
     const userItems = page.getByTestId("user-item");
     await expect(userItems).toHaveCount(1);
     await expect(userItems.first()).toContainText("Alice Johnson");
   });

   test("should show no results message when search has no matches", async ({
     page,
   }) => {
     await page.goto("/");

     await expect(page.getByTestId("user-list")).toBeVisible();

     await page.getByTestId("search-input").fill("zzzzz");

     await expect(page.getByTestId("no-results")).toBeVisible();
     await expect(page.getByTestId("user-item")).toHaveCount(0);
   });
   ```

3. Run the tests again:
   ```bash
   bun run test:e2e
   ```

**Notice:** We're keeping E2E tests focused on the happy path and critical user flows. We're not testing every edge case here — that's what integration tests are for.

---

## Part 3: Integration Testing with MSW

Integration tests render your React components and test their behavior, but instead of needing a running server, we intercept API calls using **Mock Service Worker (MSW)**. This lets us test loading states, error states, empty responses, and more — without a real server.

### Why MSW?

| Approach | Problem |
|---|---|
| Mock `fetch` directly | Fragile, doesn't test actual request/response shapes |
| Spin up a test server | Slow, complex setup |
| **MSW** | Intercepts requests at the network level, works like a real API |

MSW intercepts HTTP requests and returns mock responses. Your component code doesn't know the difference — it uses `fetch` normally, but MSW catches the request and responds with what you configure.

### Exercise 6: Install MSW and Vitest

1. Install the dependencies:
   ```bash
   bun add -d msw vitest @vitejs/plugin-react @testing-library/react @testing-library/jest-dom @testing-library/user-event happy-dom
   ```

   **What each package does:**
   - `msw` — Mock Service Worker, intercepts network requests
   - `vitest` — Test runner (you know this from Week 3)
   - `@vitejs/plugin-react` — Lets Vitest understand JSX/React components
   - `@testing-library/react` — Renders React components for testing
   - `@testing-library/jest-dom` — Extra DOM matchers like `toBeInTheDocument()`
   - `@testing-library/user-event` — Simulates user interactions (typing, clicking) in tests
   - `happy-dom` — Fast, lightweight DOM implementation used by Vitest as the test environment

2. Create `vitest.config.ts` in the project root:

   ```typescript
   import { defineConfig } from "vitest/config";
   import react from "@vitejs/plugin-react";
   import path from "path";

   export default defineConfig({
     plugins: [react()],
     test: {
       environment: "happy-dom",
       setupFiles: ["./integration/setup.ts"],
       include: ["integration/**/*.test.{ts,tsx}"],
     },
     resolve: {
       alias: {
         "@": path.resolve(__dirname, "./src"),
       },
     },
   });
   ```

   **Why `happy-dom`?** It's a fast, lightweight DOM implementation that works well with Vitest. It's faster than `jsdom` and provides the DOM APIs your React components need.

3. Create the test directories:
   ```bash
   mkdir integration mocks
   ```

4. Add a script to `package.json`:
   ```json
   {
     "scripts": {
       "test:integration": "vitest run --config vitest.config.ts",
       "test:integration:watch": "vitest --config vitest.config.ts"
     }
   }
   ```

### Exercise 7: Set Up MSW Mock Handlers

1. Create `mocks/handlers.ts` — this is where you define how your mock API responds:

   ```typescript
   import { http, HttpResponse } from "msw";

   export const mockUsers = [
     { id: 1, name: "Alice Johnson", email: "alice@example.com" },
     { id: 2, name: "Bob Smith", email: "bob@example.com" },
     { id: 3, name: "Charlie Brown", email: "charlie@example.com" },
   ];

   export const handlers = [
     // Mock GET /api/users
     http.get("/api/users", () => {
       return HttpResponse.json(mockUsers);
     }),
   ];
   ```

   **How MSW works:**
   - `http.get("/api/users", ...)` — Intercepts GET requests to `/api/users`
   - `HttpResponse.json(data)` — Returns a JSON response with the given data
   - The handler looks like a real API route, making it easy to reason about

2. Create `mocks/server.ts` — sets up the MSW server for tests:

   ```typescript
   import { setupServer } from "msw/node";
   import { handlers } from "./handlers";

   export const server = setupServer(...handlers);
   ```

3. Create `integration/setup.ts` — starts MSW before tests and cleans up after:

   ```typescript
   import { beforeAll, afterEach, afterAll } from "vitest";
   import "@testing-library/jest-dom/vitest";
   import { cleanup } from "@testing-library/react";
   import { server } from "../mocks/server";

   // Start the MSW server before all tests
   beforeAll(() => server.listen({ onUnhandledRequest: "error" }));

   // Reset handlers and clean up DOM between tests (so tests don't affect each other)
   afterEach(() => {
     cleanup();
     server.resetHandlers();
   });

   // Clean up after all tests
   afterAll(() => server.close());
   ```

   **Why `onUnhandledRequest: "error"`?** If your component makes a request you forgot to mock, the test will fail with a clear error instead of silently hanging.

   **Why `cleanup()`?** With `happy-dom`, the DOM is not automatically cleaned up between tests. Without this, rendered components from a previous test can leak into the next one, causing unexpected failures (like finding duplicate elements).

### Exercise 8: Write Integration Tests — Happy Path

1. Create `integration/UserList.test.tsx`:

   ```tsx
   import { render, screen, waitFor } from "@testing-library/react";
   import { describe, it, expect } from "vitest";
   import UserList from "@/app/components/UserList";

   describe("UserList", () => {
     it("should show loading state initially", () => {
       render(<UserList />);

       expect(screen.getByTestId("loading")).toBeInTheDocument();
       expect(screen.getByText("Loading users...")).toBeInTheDocument();
     });

     it("should display users after loading", async () => {
       render(<UserList />);

       // Wait for the user list to appear
       await waitFor(() => {
         expect(screen.getByTestId("user-list")).toBeInTheDocument();
       });

       // Check all users are rendered
       const userItems = screen.getAllByTestId("user-item");
       expect(userItems).toHaveLength(3);

       expect(screen.getByText(/Alice Johnson/)).toBeInTheDocument();
       expect(screen.getByText(/Bob Smith/)).toBeInTheDocument();
       expect(screen.getByText(/Charlie Brown/)).toBeInTheDocument();
     });
   });
   ```

2. Run the integration tests:
   ```bash
   bun run test:integration
   ```

### Exercise 9: Write Integration Tests — Error States

This is where integration tests really shine. We can easily simulate API failures that would be hard to reproduce in E2E tests.

1. Add error state tests to `integration/UserList.test.tsx`:

   First, add these imports **at the top of the file** (alongside the existing imports):

   ```tsx
   import { http, HttpResponse } from "msw";
   import { server } from "../mocks/server";
   ```

   Then add these tests **inside the existing `describe("UserList")` block**:

   ```tsx
   describe("error handling", () => {
     it("should display an error when the API fails", async () => {
       // Override the default handler for this test only
       server.use(
         http.get("/api/users", () => {
           return new HttpResponse(null, { status: 500 });
         })
       );

       render(<UserList />);

       await waitFor(() => {
         expect(screen.getByTestId("error")).toBeInTheDocument();
       });

       expect(screen.getByText(/Failed to fetch users/)).toBeInTheDocument();
     });

     it("should display an error on network failure", async () => {
       server.use(
         http.get("/api/users", () => {
           return HttpResponse.error();
         })
       );

       render(<UserList />);

       await waitFor(() => {
         expect(screen.getByTestId("error")).toBeInTheDocument();
       });
     });
   });
   ```

   **Key concept:** `server.use()` lets you override handlers for a single test. After the test, `server.resetHandlers()` in the setup file restores the defaults. This means each test is independent.

2. Run the tests:
   ```bash
   bun run test:integration
   ```

### Exercise 10: Write Integration Tests — Edge Cases

1. Add more tests for edge cases to `integration/UserList.test.tsx`:

   ```tsx
   describe("edge cases", () => {
     it("should handle an empty user list", async () => {
       server.use(
         http.get("/api/users", () => {
           return HttpResponse.json([]);
         })
       );

       render(<UserList />);

       await waitFor(() => {
         expect(screen.getByTestId("user-list")).toBeInTheDocument();
       });

       expect(screen.queryAllByTestId("user-item")).toHaveLength(0);
     });

     it("should handle a single user", async () => {
       server.use(
         http.get("/api/users", () => {
           return HttpResponse.json([
             { id: 1, name: "Only User", email: "only@example.com" },
           ]);
         })
       );

       render(<UserList />);

       await waitFor(() => {
         expect(screen.getByTestId("user-list")).toBeInTheDocument();
       });

       expect(screen.getAllByTestId("user-item")).toHaveLength(1);
       expect(screen.getByText(/Only User/)).toBeInTheDocument();
     });
   });
   ```

### Exercise 11: Test the Search Feature

1. Add search tests to `integration/UserList.test.tsx`:

   First, add this import **at the top of the file** (alongside the existing imports):

   ```tsx
   import userEvent from "@testing-library/user-event";
   ```

   Then add these tests **inside the existing `describe("UserList")` block**:

   ```tsx
   describe("search functionality", () => {
     it("should filter users by name", async () => {
       const user = userEvent.setup();

       render(<UserList />);

       await waitFor(() => {
         expect(screen.getByTestId("user-list")).toBeInTheDocument();
       });

       await user.type(screen.getByTestId("search-input"), "Alice");

       expect(screen.getAllByTestId("user-item")).toHaveLength(1);
       expect(screen.getByText(/Alice Johnson/)).toBeInTheDocument();
     });

     it("should filter users by email", async () => {
       const user = userEvent.setup();

       render(<UserList />);

       await waitFor(() => {
         expect(screen.getByTestId("user-list")).toBeInTheDocument();
       });

       await user.type(screen.getByTestId("search-input"), "bob@");

       expect(screen.getAllByTestId("user-item")).toHaveLength(1);
       expect(screen.getByText(/Bob Smith/)).toBeInTheDocument();
     });

     it("should show no results message when nothing matches", async () => {
       const user = userEvent.setup();

       render(<UserList />);

       await waitFor(() => {
         expect(screen.getByTestId("user-list")).toBeInTheDocument();
       });

       await user.type(screen.getByTestId("search-input"), "zzzzz");

       expect(screen.queryAllByTestId("user-item")).toHaveLength(0);
       expect(screen.getByTestId("no-results")).toBeInTheDocument();
     });

     it("should be case-insensitive", async () => {
       const user = userEvent.setup();

       render(<UserList />);

       await waitFor(() => {
         expect(screen.getByTestId("user-list")).toBeInTheDocument();
       });

       await user.type(screen.getByTestId("search-input"), "alice");

       expect(screen.getAllByTestId("user-item")).toHaveLength(1);
     });
   });
   ```

2. Run all integration tests:
   ```bash
   bun run test:integration
   ```

**Notice the difference:** We wrote 4 E2E tests (critical paths in a real browser) but 10+ integration tests (every edge case with mocked APIs). This is the right balance.

---

## Part 4: Running Both Test Suites

### Exercise 12: Add a Combined Test Script

1. Add a combined script to `package.json`:
   ```json
   {
     "scripts": {
       "test:e2e": "bunx playwright test",
       "test:integration": "vitest run --config vitest.config.ts",
       "test:integration:watch": "vitest --config vitest.config.ts",
       "test": "vitest run --config vitest.config.ts && bunx playwright test"
     }
   }
   ```

2. Run everything:
   ```bash
   bun run test
   ```

   **Important:** Use `bun run test`, not `bun test`. The bare `bun test` command invokes Bun's built-in test runner instead of the npm script, which will fail.

   Integration tests run first (fast), then E2E tests run (slower). If integration tests fail, you get fast feedback without waiting for E2E.

### Exercise 13: Add Tests to CI (GitHub Actions)

If you have a GitHub Actions workflow (from Week 4), add both test suites:

```yaml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: oven-sh/setup-bun@v2

      - name: Install dependencies
        run: bun install

      - name: Run integration tests
        run: bun run test:integration

      - name: Install Playwright browsers
        run: bunx playwright install --with-deps

      - name: Run E2E tests
        run: bun run test:e2e
```

---

## Part 5: Challenge Exercises

### Challenge 1: Add a Delete Button

1. Add a delete button to each user in the `UserList` component
2. Create a `DELETE /api/users/:id` API route
3. Write an E2E test that deletes a user and verifies they disappear
4. Write integration tests that cover: successful deletion, failed deletion (server error), and deleting the last user

### Challenge 2: Add Pagination

1. Update the API to return paginated results (`/api/users?page=1&limit=2`)
2. Add "Next" and "Previous" buttons to the component
3. Write integration tests for: first page, navigating forward, navigating back, last page (no "Next" button)

### Challenge 3: Slow Network Simulation

1. Add a MSW handler that delays the response by 2 seconds:
   ```typescript
   import { http, HttpResponse, delay } from "msw";

   http.get("/api/users", async () => {
     await delay(2000);
     return HttpResponse.json(mockUsers);
   });
   ```
2. Write a test that verifies the loading state persists during the delay
3. Write a Playwright test with `page.route()` to simulate a slow network

---

## Quick Reference

### Playwright — Finding Elements

| Method | When to Use |
|---|---|
| `page.getByRole("button", { name: "Submit" })` | Best — uses accessible roles |
| `page.getByTestId("user-list")` | Good — stable test IDs |
| `page.getByText("Hello")` | OK — when text is unique |
| `page.locator(".my-class")` | Avoid — breaks when CSS changes |

### Playwright — Common Assertions

| Assertion | Description |
|---|---|
| `expect(locator).toBeVisible()` | Element is visible |
| `expect(locator).toHaveCount(n)` | Number of matching elements |
| `expect(locator).toContainText("...")` | Element contains text |
| `expect(locator).toHaveValue("...")` | Input has value |
| `expect(page).toHaveURL("...")` | Page URL matches |

### MSW — Defining Handlers

```typescript
import { http, HttpResponse, delay } from "msw";

// Successful response
http.get("/api/users", () => {
  return HttpResponse.json([{ id: 1, name: "Alice" }]);
});

// Error response
http.get("/api/users", () => {
  return new HttpResponse(null, { status: 500 });
});

// Network error
http.get("/api/users", () => {
  return HttpResponse.error();
});

// Delayed response
http.get("/api/users", async () => {
  await delay(1000);
  return HttpResponse.json([]);
});
```

### Testing Library — Common Queries

| Query | Returns | Throws if not found? |
|---|---|---|
| `getByTestId("id")` | Single element | Yes |
| `queryByTestId("id")` | Single element or `null` | No |
| `getAllByTestId("id")` | Array of elements | Yes (if empty) |
| `queryAllByTestId("id")` | Array (can be empty) | No |
| `findByTestId("id")` | Promise (waits for element) | Yes |

### Common Commands

| Command | Description |
|---|---|
| `bun run test:e2e` | Run Playwright E2E tests |
| `bun run test:integration` | Run Vitest integration tests |
| `bun run test:integration:watch` | Run integration tests in watch mode |
| `bun run test` | Run all tests |
| `bunx playwright test --ui` | Open Playwright's visual test runner |
| `bunx playwright show-report` | View the HTML test report |

---

## Next Steps

After completing these exercises, explore:
- **Playwright visual regression testing** — Screenshot comparisons to catch visual changes
- **MSW with browser** — Use MSW in the browser during development to work without a backend
- **Playwright component testing** — Test individual components in a real browser (alternative to Testing Library)
- **Test coverage** — Use `vitest --coverage` and Playwright's coverage features
- **API testing with Playwright** — Use `request` context to test your API endpoints directly

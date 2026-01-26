# Setting Up a Basic TypeScript Project with Vitest and Bun

## Objective

Learn to set up a TypeScript project with Bun, implement functions using Test-Driven Development (TDD), and write comprehensive unit tests using Vitest.

## Why Test-Driven Development?

**TDD** is a development approach where you write tests *before* writing the actual code. This might seem backwards, but it offers significant benefits:

1. **Forces you to think about requirements first** - Before coding, you must understand what the function should do
2. **Catches bugs early** - Tests exist before the code, so bugs are caught immediately
3. **Provides documentation** - Tests describe how your code should behave
4. **Enables confident refactoring** - Change code knowing tests will catch any breakages
5. **Leads to better design** - Code written to be testable is usually cleaner and more modular

### The TDD Cycle: Red-Green-Refactor

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│    ┌───────┐      ┌───────┐      ┌──────────┐          │
│    │  RED  │ ───▶ │ GREEN │ ───▶ │ REFACTOR │ ───┐     │
│    └───────┘      └───────┘      └──────────┘    │     │
│        ▲                                         │     │
│        └─────────────────────────────────────────┘     │
│                                                         │
└─────────────────────────────────────────────────────────┘

RED:      Write a failing test
GREEN:    Write minimum code to pass the test
REFACTOR: Improve code quality while keeping tests green
```

## Prerequisites

- Basic knowledge of JavaScript/TypeScript
- Bun installed on your machine (https://bun.sh/)

---

## Part 1: Project Setup

### Step 1: Initialize a New Bun Project

1. Create a new directory for your project and navigate into it.
   ```bash
   mkdir week-3-vitest
   cd week-3-vitest
   ```

2. Initialize a new Bun project.
   ```bash
   bun init
   ```
   Follow the prompts to create a TypeScript-based project.

3. Install Vitest for testing.
   ```bash
   bun add -d vitest
   ```

### Step 2: Add Test Script to package.json

**Why?** Adding a script makes it easy to run tests with a simple command and ensures everyone on the team runs tests the same way.

Open `package.json` and add a test script:

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run"
  }
}
```

- `bun test` - Runs tests in watch mode (re-runs on file changes)
- `bun run test:run` - Runs tests once and exits

### Step 3: Create Project Structure

```bash
mkdir src test
```

Your project structure should look like:
```
week-3-vitest/
├── src/           # Source code
├── test/          # Test files
├── package.json
└── tsconfig.json
```

---

## Part 2: Your First TDD Cycle

### Exercise 1: The `add` Function

Let's walk through a complete TDD cycle with a simple example.

#### Step 1: Write the Test First (RED)

**Why write tests first?** It forces you to think about *what* the function should do before thinking about *how* to implement it.

Create `test/add.test.ts`:

```typescript
import { describe, it, expect } from "vitest";
import { add } from "../src/add";

describe("add", () => {
  it("should add two positive numbers", () => {
    expect(add(2, 3)).toBe(5);
  });
});
```

**Test naming conventions:**
- Use `describe` to group related tests (usually by function or feature)
- Test names should describe the expected behavior: `"should [do something] when [condition]"`
- Be specific: `"should add two positive numbers"` is better than `"works"`

Run the test:
```bash
bun test
```

The test fails because `add` doesn't exist yet. This is the **RED** phase.

#### Step 2: Write Minimum Code to Pass (GREEN)

Create `src/add.ts`:

```typescript
export function add(a: number, b: number): number {
  return a + b;
}
```

Run the test again:
```bash
bun test
```

The test passes! This is the **GREEN** phase.

#### Step 3: Add More Tests for Edge Cases

**Why test edge cases?** Real-world inputs aren't always "happy path" values. Your code should handle unusual inputs correctly.

Expand `test/add.test.ts`:

```typescript
import { describe, it, expect } from "vitest";
import { add } from "../src/add";

describe("add", () => {
  it("should add two positive numbers", () => {
    expect(add(2, 3)).toBe(5);
  });

  it("should handle negative numbers", () => {
    expect(add(-1, -2)).toBe(-3);
    expect(add(-1, 5)).toBe(4);
  });

  it("should handle zero", () => {
    expect(add(0, 5)).toBe(5);
    expect(add(5, 0)).toBe(5);
    expect(add(0, 0)).toBe(0);
  });

  it("should handle decimal numbers", () => {
    expect(add(0.1, 0.2)).toBeCloseTo(0.3); // Note: toBeCloseTo for floating point
  });

  it("should handle large numbers", () => {
    expect(add(1000000, 2000000)).toBe(3000000);
  });
});
```

**Note:** We use `toBeCloseTo` for floating-point numbers because `0.1 + 0.2` in JavaScript equals `0.30000000000000004` due to floating-point precision.

---

## Part 3: A More Realistic Example

### Exercise 2: Building a `StringCalculator`

Let's build something more practical using TDD - a string calculator that parses and adds numbers from a string.

#### Step 1: Start with the Simplest Case

Create `test/stringCalculator.test.ts`:

```typescript
import { describe, it, expect } from "vitest";
import { stringCalculator } from "../src/stringCalculator";

describe("stringCalculator", () => {
  it("should return 0 for an empty string", () => {
    expect(stringCalculator("")).toBe(0);
  });
});
```

Run tests (RED):
```bash
bun test
```

Create `src/stringCalculator.ts`:

```typescript
export function stringCalculator(input: string): number {
  return 0;
}
```

Run tests (GREEN):
```bash
bun test
```

#### Step 2: Handle a Single Number

Add a new test:

```typescript
it("should return the number for a single number string", () => {
  expect(stringCalculator("5")).toBe(5);
});
```

Run tests (RED). Update the function:

```typescript
export function stringCalculator(input: string): number {
  if (input === "") return 0;
  return parseInt(input, 10);
}
```

Run tests (GREEN).

#### Step 3: Handle Two Numbers

Add a new test:

```typescript
it("should add two comma-separated numbers", () => {
  expect(stringCalculator("1,2")).toBe(3);
});
```

Run tests (RED). Update the function:

```typescript
export function stringCalculator(input: string): number {
  if (input === "") return 0;

  const numbers = input.split(",").map(n => parseInt(n, 10));
  return numbers.reduce((sum, num) => sum + num, 0);
}
```

Run tests (GREEN).

#### Step 4: Handle Multiple Numbers

Add a new test:

```typescript
it("should add multiple comma-separated numbers", () => {
  expect(stringCalculator("1,2,3,4,5")).toBe(15);
});
```

This test should already pass! Our implementation handles any number of values.

#### Step 5: Handle Newlines as Delimiters

Add a new test:

```typescript
it("should handle newlines as delimiters", () => {
  expect(stringCalculator("1\n2,3")).toBe(6);
});
```

Run tests (RED). Update the function:

```typescript
export function stringCalculator(input: string): number {
  if (input === "") return 0;

  // Replace newlines with commas, then split
  const normalized = input.replace(/\n/g, ",");
  const numbers = normalized.split(",").map(n => parseInt(n, 10));
  return numbers.reduce((sum, num) => sum + num, 0);
}
```

Run tests (GREEN).

#### Complete Test File

Here's the complete test file for reference:

```typescript
import { describe, it, expect } from "vitest";
import { stringCalculator } from "../src/stringCalculator";

describe("stringCalculator", () => {
  describe("basic functionality", () => {
    it("should return 0 for an empty string", () => {
      expect(stringCalculator("")).toBe(0);
    });

    it("should return the number for a single number string", () => {
      expect(stringCalculator("5")).toBe(5);
    });

    it("should add two comma-separated numbers", () => {
      expect(stringCalculator("1,2")).toBe(3);
    });

    it("should add multiple comma-separated numbers", () => {
      expect(stringCalculator("1,2,3,4,5")).toBe(15);
    });
  });

  describe("delimiter handling", () => {
    it("should handle newlines as delimiters", () => {
      expect(stringCalculator("1\n2,3")).toBe(6);
    });
  });
});
```

**Note:** You can nest `describe` blocks to organize tests into logical groups.

---

## Part 4: Testing Errors and Exceptions

### Exercise 3: Input Validation

**Why test error handling?** Your functions should fail gracefully with clear error messages when given invalid input.

#### Step 1: Add Negative Number Validation

Add to `test/stringCalculator.test.ts`:

```typescript
describe("error handling", () => {
  it("should throw an error for negative numbers", () => {
    expect(() => stringCalculator("-1,2")).toThrow("Negative numbers not allowed: -1");
  });

  it("should list all negative numbers in the error message", () => {
    expect(() => stringCalculator("-1,-2,3")).toThrow("Negative numbers not allowed: -1, -2");
  });
});
```

**Note:** When testing that a function throws, wrap it in an arrow function: `expect(() => fn()).toThrow()`

Update `src/stringCalculator.ts`:

```typescript
export function stringCalculator(input: string): number {
  if (input === "") return 0;

  const normalized = input.replace(/\n/g, ",");
  const numbers = normalized.split(",").map(n => parseInt(n, 10));

  // Check for negative numbers
  const negatives = numbers.filter(n => n < 0);
  if (negatives.length > 0) {
    throw new Error(`Negative numbers not allowed: ${negatives.join(", ")}`);
  }

  return numbers.reduce((sum, num) => sum + num, 0);
}
```

---

## Part 5: Common Test Matchers

Vitest provides many matchers for different scenarios. Here are the most useful ones:

### Equality Matchers

```typescript
// Exact equality (use for primitives)
expect(2 + 2).toBe(4);

// Deep equality (use for objects and arrays)
expect({ name: "John" }).toEqual({ name: "John" });

// Floating point (avoids precision issues)
expect(0.1 + 0.2).toBeCloseTo(0.3);
```

### Truthiness Matchers

```typescript
expect(null).toBeNull();
expect(undefined).toBeUndefined();
expect("hello").toBeDefined();
expect(true).toBeTruthy();
expect(false).toBeFalsy();
```

### Number Matchers

```typescript
expect(10).toBeGreaterThan(5);
expect(10).toBeGreaterThanOrEqual(10);
expect(5).toBeLessThan(10);
expect(5).toBeLessThanOrEqual(5);
```

### String Matchers

```typescript
expect("hello world").toContain("world");
expect("hello world").toMatch(/world$/);
```

### Array Matchers

```typescript
expect([1, 2, 3]).toContain(2);
expect([1, 2, 3]).toHaveLength(3);
expect(["apple", "banana"]).toContain("banana");
```

### Exception Matchers

```typescript
// Check that it throws
expect(() => throwingFunction()).toThrow();

// Check the error message
expect(() => throwingFunction()).toThrow("specific message");

// Check error type
expect(() => throwingFunction()).toThrow(TypeError);
```

---

## Part 6: Testing Async Code

### Exercise 4: Async Functions

**Why async testing matters:** Modern applications are full of async operations - API calls, database queries, file operations. You need to know how to test them.

Create `src/userService.ts`:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// Simulates fetching a user from a database
export async function fetchUser(id: number): Promise<User> {
  // Simulate network delay
  await new Promise(resolve => setTimeout(resolve, 100));

  if (id <= 0) {
    throw new Error("Invalid user ID");
  }

  return {
    id,
    name: `User ${id}`,
    email: `user${id}@example.com`
  };
}

export async function fetchUsers(ids: number[]): Promise<User[]> {
  const users = await Promise.all(ids.map(id => fetchUser(id)));
  return users;
}
```

Create `test/userService.test.ts`:

```typescript
import { describe, it, expect } from "vitest";
import { fetchUser, fetchUsers } from "../src/userService";

describe("userService", () => {
  describe("fetchUser", () => {
    it("should return a user with the correct id", async () => {
      const user = await fetchUser(1);

      expect(user.id).toBe(1);
      expect(user.name).toBe("User 1");
      expect(user.email).toBe("user1@example.com");
    });

    it("should throw an error for invalid user ID", async () => {
      await expect(fetchUser(0)).rejects.toThrow("Invalid user ID");
      await expect(fetchUser(-1)).rejects.toThrow("Invalid user ID");
    });
  });

  describe("fetchUsers", () => {
    it("should return multiple users", async () => {
      const users = await fetchUsers([1, 2, 3]);

      expect(users).toHaveLength(3);
      expect(users[0].id).toBe(1);
      expect(users[2].id).toBe(3);
    });

    it("should return an empty array for empty input", async () => {
      const users = await fetchUsers([]);

      expect(users).toEqual([]);
    });
  });
});
```

**Key points for async testing:**
- Use `async/await` in your test functions
- For rejected promises, use `await expect(...).rejects.toThrow()`
- Tests wait for async operations to complete before assertions

---

## Part 7: Challenge Exercises

### Challenge 1: FizzBuzz with TDD

Build the classic FizzBuzz function using TDD:
- Returns "Fizz" for multiples of 3
- Returns "Buzz" for multiples of 5
- Returns "FizzBuzz" for multiples of both 3 and 5
- Returns the number as a string otherwise

Start with the simplest test and build up:

```typescript
describe("fizzBuzz", () => {
  it("should return the number as string for non-multiples", () => {
    expect(fizzBuzz(1)).toBe("1");
    expect(fizzBuzz(2)).toBe("2");
  });

  // Add more tests...
});
```

### Challenge 2: Password Validator

Build a password validator with these rules:
- At least 8 characters
- Contains at least one uppercase letter
- Contains at least one lowercase letter
- Contains at least one number
- Contains at least one special character (!@#$%^&*)

The function should return `{ valid: boolean, errors: string[] }`.

### Challenge 3: Array Utilities

Build a utility module with these functions using TDD:
- `unique(arr)` - Returns array with duplicates removed
- `flatten(arr)` - Flattens nested arrays
- `chunk(arr, size)` - Splits array into chunks of given size

---

## Quick Reference

### Test Structure

```typescript
import { describe, it, expect, beforeEach, afterEach } from "vitest";

describe("ModuleName", () => {
  // Runs before each test
  beforeEach(() => {
    // Setup code
  });

  // Runs after each test
  afterEach(() => {
    // Cleanup code
  });

  describe("functionName", () => {
    it("should do something when condition", () => {
      // Arrange
      const input = "test";

      // Act
      const result = functionName(input);

      // Assert
      expect(result).toBe(expected);
    });
  });
});
```

### Common Commands

| Command | Description |
|---------|-------------|
| `bun test` | Run tests in watch mode |
| `bun run test:run` | Run tests once |
| `bun test src/add` | Run tests matching pattern |
| `bun test --reporter=verbose` | Detailed test output |

### Test Naming Best Practices

```typescript
// ❌ Bad - vague
it("works", () => {});
it("test add", () => {});

// ✅ Good - descriptive
it("should return 0 when both inputs are 0", () => {});
it("should throw an error when input is negative", () => {});
it("should handle empty strings gracefully", () => {});
```

---

## Next Steps

After completing these exercises, explore:
- **Snapshot testing** - For testing component output or large data structures
- **Test fixtures** - Reusable test data
- **Parameterized tests** - `it.each()` for testing multiple inputs
- **Test doubles (mocks, stubs, spies)** - For isolating units
- **Integration testing** - Testing how modules work together

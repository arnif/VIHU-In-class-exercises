# Setting Up a Basic TypeScript Project with Vitest and Bun

## Objective

Learn to set up a TypeScript project with Bun, implement a basic function (adding two numbers), and write unit tests using Vitest, focusing on the Test-Driven Development (TDD) approach.

## Prerequisites

- Basic knowledge of JavaScript/TypeScript
- Bun installed on your machine (https://bun.sh/)

## Step-by-Step Guide

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
   bun add vitest
   ```

### Step 2: Create Your Test Directory and Files

1. Create a `test` directory in your project root.
   ```bash
   mkdir test
   ```
2. Create a test file `add.test.ts` in the `test` directory.
   ```bash
   touch test/add.test.ts
   ```

3. Add the following content to `add.test.ts`:

   ```typescript
   import { it, expect } from "vitest";
   import { add } from "../src/add";

   it("adds two numbers", () => {
     expect(add(2, 3)).toBe(5);
   });
   ```

4. Run the test using Vitest (it will fail because the `add` function doesn't exist yet).
   ```bash
   bun vitest
   ```

### Step 3: Write the Function Incrementally

#### Step 3.1: Create the `add` Function

1. Create a `src` directory for your source files.
   ```bash
   mkdir src
   ```
2. Create a file `add.ts` in the `src` directory.
   ```bash
   touch src/add.ts
   ```
3. Add the following initial implementation of the `add` function:
   ```typescript
   export function add(a: number, b: number): number {
     return 5; // Hardcoded value to make the initial test pass
   }
   ```

#### Step 3.2: Verify the Test Passes

1. Run the test again:
   ```bash
   bun vitest
   ```
   - The test will pass because the function is hardcoded to return 5.

#### Step 3.3: Add Another Test (Red Phase)

1. Extend `add.test.ts` with another test case:
   ```typescript
   it("adds other numbers", () => {
     expect(add(4, 6)).toBe(10);
   });
   ```

2. Run the tests again:
   ```bash
   bun vitest
   ```
   - The second test will fail because the `add` function is still hardcoded to return 5.

#### Step 3.4: Update the `add` Function (Green Phase)

1. Update the `add` function in `src/add.ts` to handle any inputs:
   ```typescript
   export function add(a: number, b: number): number {
     return a + b; // Correct implementation
   }
   ```

2. Run the tests again to ensure all pass:
   ```bash
   bun vitest
   ```

#### Step 3.5: Refactor if Necessary

1. Review the code and ensure it is clean and efficient.
2. Ensure all tests pass after refactoring.

---

### Step 4: Follow the TDD Cycle

1. **Red Phase**: Write a failing test for new functionality.
2. **Green Phase**: Implement the minimum code needed to pass the test.
3. **Refactor Phase**: Optimize your code while ensuring all tests pass.
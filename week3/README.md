# Setting Up a Basic TypeScript Project with Vitest

## Objective

Learn to set up a TypeScript project from scratch, implement a basic function (adding two numbers), and write unit tests using Vitest, focusing on the Test-Driven Development (TDD) approach.

## Prerequisites

- Basic knowledge of JavaScript/TypeScript
- Node.js installed on your machine

## Step-by-Step Guide

### Step 1: Initialize a New Node.js Project

1. Create a new directory for your project and navigate into it.
   ```bash
   mkdir week-3-vitest
   cd week-3-vitest
   ```
2. Initialize a new Node.js project.
   ```bash
   npm init -y
   ```

### Step 2: Install TypeScript and Vitest

1. Install TypeScript as a dependency.
   ```bash
   npm install typescript --save
   ```
2. Install Vitest for testing.
   ```bash
   npm install vitest --save
   ```

### Step 3: Configure TypeScript and Vitest

1. Create a `tsconfig.json` file for TypeScript configuration.
   ```bash
   npx tsc --init
   ```
   
2. Modify the `tsconfig.json` as needed for your project.:
   ```json
   "compilerOptions": {
        "target": "ES2022",
        "module": "ES2022",
        "moduleResolution": "node",
        "outDir": "dist",
        "strict": true,
        "skipLibCheck": true
    }
   ```

### Step 4: Implementing TDD with a Basic Function

1. **Write the Test First (Red Phase)**:

   - Create a test file `add.test.ts` in a `test` directory.
   - Write a simple test case expecting the `add` function to correctly add two numbers.

     ```typescript
     import { it, expect } from "vitest";
     import { add } from "../src/add";

     it("adds two numbers", () => {
       expect(add(2, 3)).toBe(5);
     });
     ```

2. **Run the Test and See it Fail**:
   - Run Vitest to see the test fail (as we haven't implemented `add` yet).
     ```bash
     npx vitest
     ```

### Step 5: Implement the Function (Green Phase)

1. Create a file `add.ts` in a `src` directory.
2. Implement the `add` function.
   ```typescript
   export function add(a: number, b: number): number {
     return 5;
   }
   ```
3. Run the test again and see it pass.

### Step 6: Refactor if Needed (Refactor Phase)

1. Review your `add` function and tests.
2. Make any necessary refactoring to improve the code quality without changing its functionality.
3. Run the tests again to ensure everything still works.

### Step 7: Final Steps

1. Repeat the TDD cycle (Red, Green, Refactor) for more complex functionalities.
2. Use Git for version control to track changes and maintain a history of your project.

# Code Style: Linting and Formatting with Biome

## Objective

Learn how to enforce consistent code style in a TypeScript project using Biome — a single tool that handles both linting and formatting. By the end, you'll have automated tooling that catches problems and formats your code.

---

## Why Code Style Matters

Code style isn't just personal preference. It's about writing code that is **clean, readable, and maintainable** for everyone on the team.

### Key Principles

| Principle | Why It Matters |
|---|---|
| **Consistency** | Same patterns everywhere make code predictable |
| **Readability** | Others (and future you) can understand the code quickly |
| **Maintainability** | Consistent code is easier to change and debug |

### Why Automate It?

| Manual Code Style | Automated Code Style |
|---|---|
| Tedious and time-consuming | Formatted in the blink of an eye |
| Prone to human error | Consistent every time |
| Style debates in code reviews | No need to discuss style |
| "I forgot to format" | Happens automatically on save |
| Different styles per developer | One standard for the whole team |

---

## Why Biome?

Traditionally, JavaScript/TypeScript projects use two separate tools: **ESLint** for linting (finding bugs) and **Prettier** for formatting (fixing style). The problem is that these tools can conflict with each other and require extra configuration to play nicely together.

**Biome** is a single tool that handles both linting and formatting with zero conflict. It's also written in Rust, making it significantly faster.

| ESLint + Prettier | Biome |
|---|---|
| Two tools to install and configure | One tool does both |
| Can conflict with each other | No conflicts by design |
| Requires `eslint-config-prettier` to avoid clashes | Not needed |
| Slower (JavaScript-based) | Much faster (Rust-based) |
| Two config files | One config file |

---

## Prerequisites

- **Bun** installed (https://bun.sh/)
- A code editor (VS Code recommended)
- Basic knowledge of TypeScript

---

## Part 1: Project Setup

### Exercise 1: Initialize a Bun Project

1. Create a new project folder and initialize it:
   ```bash
   mkdir code-style-exercise
   cd code-style-exercise
   bun init
   ```
   Follow the prompts to create a TypeScript-based project.

2. Create a source directory:
   ```bash
   mkdir src
   ```

3. Your project structure should look like:
   ```
   code-style-exercise/
   ├── src/
   ├── package.json
   └── tsconfig.json
   ```

---

## Part 2: Setting Up Biome

### Exercise 2: Install and Initialize Biome

1. Install Biome:
   ```bash
   bun add -d --exact @biomejs/biome
   ```

2. Initialize Biome configuration:
   ```bash
   bunx @biomejs/biome init
   ```
   This creates a `biome.json` file in the project root.

3. Open `biome.json` and review the default configuration. Update it to:
   ```json
   {
     "$schema": "https://biomejs.dev/schemas/2.0.0/schema.json",
     "organizeImports": {
       "enabled": true
     },
     "formatter": {
       "enabled": true,
       "indentStyle": "space",
       "indentWidth": 2,
       "lineWidth": 80
     },
     "linter": {
       "enabled": true,
       "rules": {
         "recommended": true
       }
     },
     "files": {
       "ignore": ["node_modules", "dist"]
     }
   }
   ```

   **What these options mean:**

   **Formatter settings:**
   - `indentStyle`: Use spaces (or `"tab"` for tabs)
   - `indentWidth`: 2 spaces per indent level
   - `lineWidth`: Wrap lines at 80 characters

   **Linter settings:**
   - `recommended`: Enables a curated set of rules that catch common bugs and enforce best practices

4. Add scripts to your `package.json`:
   ```json
   {
     "scripts": {
       "format": "biome format --write src/",
       "lint": "biome lint src/",
       "check": "biome check src/",
       "fix": "biome check --write src/"
     }
   }
   ```

   **What these commands do:**
   - `format` — Formats files (fixes style)
   - `lint` — Checks for code problems (reports only)
   - `check` — Runs both formatting check and linting in one command
   - `fix` — Runs both formatting and linting with auto-fix

---

## Part 3: Formatting with Biome

### Exercise 3: See Formatting in Action

1. Create a file `src/messy.ts` with intentionally bad formatting:
   ```typescript
   const greeting="Hello, World!"
   const   add = (a:number,    b:number):number => {
       return a+b
   }
   const users = [{name: "Alice",age: 30},{name: "Bob",age: 25},{name: "Charlie",age: 35}]
   console.log(greeting)
   console.log(add(1,2))
   console.log(users)
   ```

2. Run Biome format in check mode to see what it would change:
   ```bash
   bunx @biomejs/biome format src/
   ```
   Biome will show the formatted output without modifying the file.

3. Run Biome to format the file:
   ```bash
   bun run format
   ```

4. Open `src/messy.ts` again and observe the changes. The file should now be properly formatted with consistent spacing, semicolons, and line breaks.

### Exercise 4: Experiment with Formatter Options

1. Try changing the formatter options in `biome.json`:
   ```json
   {
     "formatter": {
       "enabled": true,
       "indentStyle": "tab",
       "lineWidth": 100
     }
   }
   ```

2. Run the formatter again and observe how the output changes:
   ```bash
   bun run format
   ```

3. **Revert** back to the original settings before continuing:
   ```json
   {
     "formatter": {
       "enabled": true,
       "indentStyle": "space",
       "indentWidth": 2,
       "lineWidth": 80
     }
   }
   ```

4. Format again to apply the reverted settings:
   ```bash
   bun run format
   ```

---

## Part 4: Linting with Biome

Formatting fixes how code looks. Linting finds actual problems — bugs, unused code, unsafe patterns, and more.

### Exercise 5: See Linting in Action

1. Create a file `src/buggy.ts` with intentional problems:
   ```typescript
   let greeting = "Hello, World!";

   let unusedVariable = 42;

   function isEqual(a: number, b: number): boolean {
     if (a == b) {
       return true;
     }
     return false;
   }

   let result = isEqual(1, 1);
   console.log(result);
   console.log(greeting);
   ```

2. Run the linter to find the problems:
   ```bash
   bun run lint
   ```

3. You should see errors for:
   - `unusedVariable` is declared but never used
   - `==` should be `===` (use of loose equality)
   - `let` should be `const` for variables that are never reassigned

4. Run Biome with auto-fix to fix what it can:
   ```bash
   bun run fix
   ```

5. Open `src/buggy.ts` and observe:
   - `let` has been changed to `const` where appropriate
   - `==` has been changed to `===`
   - `unusedVariable` still shows a warning (Biome can't know if you meant to use it)

### Exercise 6: Configure Linter Rules

Biome organizes rules into groups. You can enable, disable, or change the severity of individual rules.

1. Update the `linter` section in `biome.json` to customize rules:
   ```json
   {
     "linter": {
       "enabled": true,
       "rules": {
         "recommended": true,
         "style": {
           "noVar": "error",
           "useConst": "error",
           "useTemplate": "warn"
         },
         "suspicious": {
           "noDoubleEquals": "error",
           "noExplicitAny": "warn"
         },
         "correctness": {
           "noUnusedVariables": "warn",
           "noUnusedImports": "error"
         }
       }
     }
   }
   ```

   **Rule groups explained:**
   - `style` — Code style preferences (e.g., `const` over `let`, template literals)
   - `suspicious` — Patterns that are likely bugs (e.g., `==` instead of `===`)
   - `correctness` — Definitely wrong code (e.g., unused variables or imports)
   - `complexity` — Overly complex code that could be simplified
   - `performance` — Code that could be more performant

   **Rule severity levels:**
   - `"off"` — Disable the rule
   - `"warn"` — Show a warning (doesn't fail the build)
   - `"error"` — Show an error (fails the build)

2. Run the linter again to see the updated output:
   ```bash
   bun run lint
   ```

---

## Part 5: Using `biome check` — The All-in-One Command

The real power of Biome is `biome check`, which runs formatting, linting, and import sorting all in one pass.

### Exercise 7: Check and Fix Everything at Once

1. Create a file `src/everything.ts` with both formatting and linting issues:
   ```typescript
   import {readFileSync} from "fs"
   import {join} from "path"

   var   name="World"
   let greeting = "Hello, " +name+ "!"

   let unused = 123

   if(greeting == "Hello, World!") {
       console.log(greeting)
   }
   ```

2. Run `biome check` to see all issues at once:
   ```bash
   bun run check
   ```

   You should see both formatting issues and lint errors reported together.

3. Fix everything in one command:
   ```bash
   bun run fix
   ```

4. Open `src/everything.ts` and observe — formatting is fixed, `var` is changed to `const`/`let`, `==` is changed to `===`, and imports are sorted.

---

## Part 6: CI Integration

Code style should be enforced in your CI pipeline. The build should fail if code doesn't meet the standards.

### Exercise 8: Add a CI Check

1. The `check` script you already have is perfect for CI:
   ```bash
   bun run check
   ```

   This exits with a non-zero code if any formatting or lint issues are found, which will fail the CI build.

2. If you have a GitHub Actions workflow (as you learned in week 4), you could add this step:
   ```yaml
   - name: Check code style
     run: bun run check
   ```

---

## Quick Reference

### Biome Formatter Options

| Option | Default | Description |
|---|---|---|
| `indentStyle` | `"tab"` | Use `"space"` or `"tab"` |
| `indentWidth` | `2` | Spaces per indent level |
| `lineWidth` | `80` | Line wrap width |
| `lineEnding` | `"lf"` | Line ending style |

### Common Biome Lint Rules

| Rule | Group | Description |
|---|---|---|
| `useConst` | style | Require `const` when never reassigned |
| `noVar` | style | Disallow `var` |
| `useTemplate` | style | Prefer template literals over string concatenation |
| `noDoubleEquals` | suspicious | Require `===` and `!==` |
| `noExplicitAny` | suspicious | Disallow the `any` type |
| `noUnusedVariables` | correctness | Disallow unused variables |
| `noUnusedImports` | correctness | Disallow unused imports |

### Common Commands

| Command | Description |
|---|---|
| `bun run format` | Format files with Biome |
| `bun run lint` | Check code for problems |
| `bun run check` | Run formatting check + linting together |
| `bun run fix` | Auto-fix all formatting and lint issues |

### The Tools Landscape

| Tool | Language | Purpose |
|---|---|---|
| **Biome** | Many languages | Linting + Formatting (all-in-one) |
| **ESLint** | JavaScript/TypeScript | Linting (find problems) |
| **Prettier** | Many languages | Formatting (fix style) |
| **Autopep8** | Python | Formatting |
| **Black** | Python | Formatting |
| **clang-format** | C/C++ | Formatting |
| **google-java-format** | Java | Formatting |

---

## Next Steps

After completing these exercises, explore:
- **Husky + lint-staged** — Run Biome automatically on git commit
- **Biome VS Code extension** — Format and lint on save in your editor
- **ESLint + Prettier** — The traditional (but more complex) alternative to Biome
- **EditorConfig** — Share basic editor settings across different editors

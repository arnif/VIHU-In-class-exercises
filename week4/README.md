# GitHub Actions: Automating Your Development Workflow

## Why CI/CD?

**Continuous Integration (CI)** and **Continuous Deployment (CD)** automate repetitive tasks in your development workflow. Instead of manually running tests, checking code quality, and deploying - these happen automatically.

### Problems CI/CD Solves

| Without CI/CD | With CI/CD |
|---------------|------------|
| "It works on my machine" | Tests run in a consistent environment |
| Forgetting to run tests before merging | Tests run automatically on every PR |
| Manual, error-prone deployments | Automated, reproducible deployments |
| Code quality varies by developer | Automated linting enforces standards |
| Hours spent on repetitive tasks | Automation handles the boring stuff |

### How GitHub Actions Works

```
┌─────────────────────────────────────────────────────────────┐
│                         GitHub                              │
│  ┌──────────┐    ┌──────────────┐    ┌─────────────────┐   │
│  │  Event   │───▶│   Workflow   │───▶│     Results     │   │
│  │  (push)  │    │  (.yml file) │    │  (pass/fail)    │   │
│  └──────────┘    └──────────────┘    └─────────────────┘   │
│                         │                                   │
│                         ▼                                   │
│                  ┌──────────────┐                          │
│                  │    Runner    │                          │
│                  │  (Ubuntu VM) │                          │
│                  └──────────────┘                          │
└─────────────────────────────────────────────────────────────┘
```

1. An **event** triggers the workflow (push, pull request, schedule, etc.)
2. GitHub reads your **workflow file** (`.github/workflows/*.yml`)
3. A **runner** (virtual machine) executes your jobs
4. **Results** appear in the Actions tab and on pull requests

---

## YAML Basics

GitHub Actions uses YAML for configuration. Here's a quick primer:

```yaml
# Comments start with #

# Key-value pairs
name: My Workflow
version: 1.0

# Nested structures (use 2-space indentation)
person:
  name: John
  age: 30

# Lists (use dashes)
fruits:
  - apple
  - banana
  - orange

# Inline list
colors: [red, green, blue]

# Multi-line strings
description: |
  This is a multi-line
  string that preserves
  line breaks.

# Boolean values
enabled: true
disabled: false
```

**Common YAML mistakes:**
- Wrong indentation (must be consistent, use spaces not tabs)
- Missing colons after keys
- Incorrect list formatting

---

## Part 1: Your First Workflow

### Exercise 1: Hello World

Let's start with the simplest possible workflow.

#### Step 1: Create the Workflow Directory

```bash
mkdir -p .github/workflows
```

#### Step 2: Create the Workflow File

Create `.github/workflows/hello.yml`:

```yaml
name: Hello World

# When to run this workflow
on: [push]

# What to do
jobs:
  greet:
    # Which virtual machine to use
    runs-on: ubuntu-latest

    steps:
      # Step 1: Print a message
      - name: Say Hello
        run: echo "Hello, GitHub Actions!"

      # Step 2: Show some info
      - name: Show Environment Info
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Triggered by: ${{ github.actor }}"
```

#### Step 3: Commit and Push

```bash
git add .github/workflows/hello.yml
git commit -m "Add hello world workflow"
git push
```

#### Step 4: View the Results

1. Go to your GitHub repository
2. Click the **Actions** tab
3. Click on the workflow run to see logs

**Understanding the workflow:**
- `name`: Display name in the Actions tab
- `on`: Events that trigger this workflow
- `jobs`: Groups of steps that run together
- `runs-on`: The virtual machine type
- `steps`: Individual commands to execute
- `${{ }}`: GitHub expression syntax for variables

---

## Part 2: Running Tests Automatically

This is where CI becomes useful! Let's run your tests from week3 automatically.

### Exercise 2: Test Workflow

Create `.github/workflows/test.yml`:

```yaml
name: Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      # Step 1: Get the code
      - name: Checkout code
        uses: actions/checkout@v4

      # Step 2: Set up Bun
      - name: Setup Bun
        uses: oven-sh/setup-bun@v2

      # Step 3: Install dependencies
      - name: Install dependencies
        run: bun install

      # Step 4: Run tests
      - name: Run tests
        run: bun test
```

**What's new here:**
- `uses`: Uses a pre-built action from the marketplace
- `actions/checkout@v4`: Clones your repository
- `oven-sh/setup-bun@v2`: Installs Bun on the runner

### Understanding Triggers

```yaml
on:
  push:
    branches: [main]      # Only on pushes to main
  pull_request:
    branches: [main]      # On PRs targeting main
```

This means:
- Direct pushes to `main` → tests run
- Pull requests to `main` → tests run
- Pushes to feature branches → tests don't run (unless there's a PR)

---

## Part 3: Workflow Triggers

GitHub Actions can be triggered by many events. Here are the most common:

### Common Triggers

```yaml
# Run on every push to any branch
on: push

# Run on push to specific branches
on:
  push:
    branches: [main, develop]

# Run on pull requests
on:
  pull_request:
    branches: [main]

# Run on both push and pull request
on: [push, pull_request]

# Run on a schedule (cron syntax)
on:
  schedule:
    - cron: '0 0 * * *'  # Every day at midnight UTC

# Manual trigger (button in Actions tab)
on:
  workflow_dispatch:
```

### Exercise 3: Add Manual Trigger

Update your test workflow to allow manual runs:

```yaml
name: Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:  # Add this line

jobs:
  test:
    # ... rest of the workflow
```

Now you can click "Run workflow" in the Actions tab to trigger it manually.

---

## Part 4: Caching Dependencies

Installing dependencies on every run is slow. Caching speeds things up significantly.

### Exercise 4: Add Caching

Update `.github/workflows/test.yml`:

```yaml
name: Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Bun
        uses: oven-sh/setup-bun@v2

      # Cache Bun dependencies
      - name: Cache dependencies
        uses: actions/cache@v4
        with:
          path: ~/.bun/install/cache
          key: ${{ runner.os }}-bun-${{ hashFiles('**/bun.lockb') }}
          restore-keys: |
            ${{ runner.os }}-bun-

      - name: Install dependencies
        run: bun install

      - name: Run tests
        run: bun test
```

**How caching works:**
1. First run: No cache exists, installs everything, saves cache
2. Second run: Cache found, restores it, `bun install` is much faster
3. When `bun.lockb` changes: New cache key, fresh install

---

## Part 5: Matrix Builds

Test your code across multiple versions or operating systems simultaneously.

### Exercise 5: Test Multiple Bun Versions

```yaml
name: Tests (Matrix)

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        bun-version: ['latest', '1.0.0']

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Bun ${{ matrix.bun-version }}
        uses: oven-sh/setup-bun@v2
        with:
          bun-version: ${{ matrix.bun-version }}

      - name: Install dependencies
        run: bun install

      - name: Run tests
        run: bun test
```

This creates **2 parallel jobs** - one for each Bun version!

### Matrix with Multiple Dimensions

```yaml
strategy:
  matrix:
    bun-version: ['latest', '1.0.0']
    os: [ubuntu-latest, macos-latest]
```

This creates **4 jobs** (2 versions × 2 operating systems).

---

## Part 6: Multiple Jobs and Dependencies

Workflows can have multiple jobs that run in parallel or sequentially.

### Exercise 6: Lint and Test Pipeline

Create `.github/workflows/ci.yml`:

```yaml
name: CI Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # Job 1: Check code quality
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Bun
        uses: oven-sh/setup-bun@v2

      - name: Install dependencies
        run: bun install

      - name: Run linter
        run: bun run lint

  # Job 2: Run tests (only if lint passes)
  test:
    runs-on: ubuntu-latest
    needs: lint  # Wait for lint job to complete

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Bun
        uses: oven-sh/setup-bun@v2

      - name: Install dependencies
        run: bun install

      - name: Run tests
        run: bun test

  # Job 3: Build (only if tests pass)
  build:
    runs-on: ubuntu-latest
    needs: test  # Wait for test job to complete

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Bun
        uses: oven-sh/setup-bun@v2

      - name: Install dependencies
        run: bun install

      - name: Build
        run: bun run build
```

**Understanding `needs`:**
- `needs: lint` → wait for `lint` to finish
- `needs: [lint, test]` → wait for both jobs
- Without `needs` → jobs run in parallel

```
Without needs:          With needs:
┌──────┐ ┌──────┐      ┌──────┐
│ lint │ │ test │      │ lint │
└──────┘ └──────┘      └───┬──┘
    (parallel)             │
                       ┌───▼──┐
                       │ test │
                       └───┬──┘
                           │
                       ┌───▼───┐
                       │ build │
                       └───────┘
                       (sequential)
```

---

## Part 7: Conditional Execution

Control when steps or jobs run using conditions.

### Exercise 7: Conditional Steps

```yaml
name: Conditional Workflow

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Bun
        uses: oven-sh/setup-bun@v2

      - name: Install dependencies
        run: bun install

      - name: Run tests
        run: bun test

      # Only run on main branch
      - name: Build for production
        if: github.ref == 'refs/heads/main'
        run: bun run build

      # Only run on pull requests
      - name: Comment on PR
        if: github.event_name == 'pull_request'
        run: echo "This is a pull request!"

      # Run even if previous steps fail
      - name: Cleanup
        if: always()
        run: echo "Cleaning up..."
```

**Common conditions:**
- `if: github.ref == 'refs/heads/main'` - Only on main branch
- `if: github.event_name == 'pull_request'` - Only on PRs
- `if: success()` - Only if all previous steps passed (default)
- `if: failure()` - Only if a previous step failed
- `if: always()` - Run regardless of previous step status

---

## Part 8: Status Badges

Add a badge to your README showing your workflow status.

### Exercise 8: Add a Status Badge

Add this to your `README.md`:

```markdown
![CI](https://github.com/YOUR_USERNAME/YOUR_REPO/actions/workflows/ci.yml/badge.svg)
```

Replace:
- `YOUR_USERNAME` with your GitHub username
- `YOUR_REPO` with your repository name
- `ci.yml` with your workflow filename

**Badge examples:**

| Status | Badge |
|--------|-------|
| Passing | ![passing](https://img.shields.io/badge/build-passing-brightgreen) |
| Failing | ![failing](https://img.shields.io/badge/build-failing-red) |

---

## Part 9: Complete CI/CD Example

Here's a complete, production-ready workflow combining everything:

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Bun
        uses: oven-sh/setup-bun@v2

      - name: Cache dependencies
        uses: actions/cache@v4
        with:
          path: ~/.bun/install/cache
          key: ${{ runner.os }}-bun-${{ hashFiles('**/bun.lockb') }}
          restore-keys: |
            ${{ runner.os }}-bun-

      - name: Install dependencies
        run: bun install

      - name: Run linter
        run: bun run lint
        continue-on-error: false

      - name: Run tests
        run: bun test

      - name: Build
        run: bun run build
        if: github.ref == 'refs/heads/main'
```

**For this to work, your `package.json` should have:**

```json
{
  "scripts": {
    "test": "vitest run",
    "lint": "echo 'Add your linter here (e.g., eslint .)' ",
    "build": "echo 'Add your build command here'"
  }
}
```

---

## Troubleshooting

### Common Issues and Solutions

#### Workflow not running

| Problem | Solution |
|---------|----------|
| File not in `.github/workflows/` | Check the directory path |
| YAML syntax error | Use a YAML validator online |
| Wrong trigger event | Check your `on:` section |
| Branch name mismatch | Ensure branch names match exactly |

#### Workflow fails

1. **Click on the failed job** in the Actions tab
2. **Expand the failed step** to see the error
3. **Check the logs** for specific error messages

#### Common errors

```yaml
# ❌ Wrong: Tabs instead of spaces
jobs:
	test:  # Using tab

# ✅ Correct: Use spaces (2-space indent)
jobs:
  test:  # Using spaces
```

```yaml
# ❌ Wrong: Missing checkout
steps:
  - name: Run tests
    run: bun test  # Files don't exist!

# ✅ Correct: Checkout first
steps:
  - uses: actions/checkout@v4
  - name: Run tests
    run: bun test
```

```yaml
# ❌ Wrong: Typo in action name
uses: actions/checkout@v4  # correct
uses: action/checkout@v4   # missing 's'

# ✅ Check spelling carefully
```

### Debugging Tips

1. **Add debug output:**
   ```yaml
   - name: Debug info
     run: |
       pwd
       ls -la
       echo "Branch: ${{ github.ref }}"
   ```

2. **Check environment:**
   ```yaml
   - name: Show environment
     run: env | sort
   ```

3. **Use workflow_dispatch for testing:**
   - Add `workflow_dispatch` to triggers
   - Manually run from Actions tab
   - Iterate quickly without pushing

---

## Quick Reference

### Workflow Structure

```yaml
name: Workflow Name        # Display name

on:                        # Triggers
  push:
    branches: [main]

env:                       # Global environment variables
  NODE_ENV: production

jobs:
  job-name:                # Job identifier
    runs-on: ubuntu-latest # Runner

    steps:
      - uses: action@version    # Use an action
      - name: Step name         # Run a command
        run: command
```

### Common Actions

| Action | Purpose |
|--------|---------|
| `actions/checkout@v4` | Clone repository |
| `actions/cache@v4` | Cache dependencies |
| `oven-sh/setup-bun@v2` | Install Bun |
| `actions/setup-node@v4` | Install Node.js |

### Useful Expressions

| Expression | Value |
|------------|-------|
| `${{ github.repository }}` | owner/repo-name |
| `${{ github.ref_name }}` | Branch or tag name |
| `${{ github.actor }}` | User who triggered |
| `${{ github.event_name }}` | push, pull_request, etc. |
| `${{ runner.os }}` | Linux, Windows, macOS |

---

## Exercises to Try

1. **Basic**: Create a workflow that prints your name and the current date

2. **Testing**: Set up a workflow that runs your week3 Vitest tests on every push

3. **Matrix**: Test your code on both `ubuntu-latest` and `macos-latest`

4. **Caching**: Add caching to speed up your test workflow

5. **Multi-job**: Create a workflow with separate lint, test, and build jobs that run in sequence

6. **Badge**: Add a status badge to your repository's README

---

## Next Steps

After completing these exercises, explore:
- **Secrets and environment variables** - Storing API keys and passwords securely
- **Deployment workflows** - Automatically deploy to hosting services
- **Reusable workflows** - Share workflows across repositories
- **Custom actions** - Build your own actions
- **Branch protection rules** - Require CI to pass before merging

# Week 12: Agentic Engineering

## Objective

Experience **agentic engineering** — the practice of building software collaboratively with an AI coding agent. You'll use **OpenCode**, an open-source AI coding agent in the terminal, to build something of your own choosing in a mini hackathon. The goal isn't just to get code written — it's to learn how the 11 weeks of knowledge you've built (HTTP servers, testing, Docker, CI/CD, databases, Kubernetes, Terraform, monitoring) make you a better **guide and reviewer** of AI-generated code.

---

## Why Agentic Engineering?

AI coding agents can write code, run tests, fix bugs, and scaffold entire projects. But they don't replace engineers — they change how engineers work.

Without knowledge, you become a **passenger**: you accept whatever the agent produces, can't tell if it's good or bad, and can't fix it when it breaks.

With knowledge, you become a **pilot**: you direct the agent, evaluate its output, catch mistakes, and course-correct. Everything you've learned in this course — how HTTP servers work, what makes a good test, how Docker containers are built, why monitoring matters — is what makes you effective with these tools.

### The Agentic Loop

Agentic engineering follows a cycle:

```
You (prompt) → Agent (generates) → You (review) → You (refine or accept)
     ↑                                                      |
     └──────────────────────────────────────────────────────┘
```

| Step | What You Do | What the Agent Does |
|---|---|---|
| **Prompt** | Describe what you want, with context and constraints | — |
| **Generate** | — | Write code, create files, run commands |
| **Review** | Read the code, check if it's correct and complete | — |
| **Refine** | Ask for changes, point out issues, add requirements | Iterate based on feedback |

The better your prompts and reviews are, the better the output. Your domain knowledge is the difference between getting useful code and getting plausible-looking garbage.

---

## What is OpenCode?

**OpenCode** is an open-source AI coding agent that runs in your terminal. It can read your project files, write code, run shell commands, and iterate on changes — all through natural language conversation.

| Feature | What It Does |
|---|---|
| Terminal TUI | Interactive chat interface in your terminal |
| File access | Reads and edits files in your project |
| Shell commands | Runs tests, installs packages, starts servers |
| Multiple providers | Works with many AI model providers |
| Free models | Several models available at no cost — no login required |
| Plan vs. Build mode | Switch between read-only planning and active code changes |

---

## Prerequisites

- **Bun** installed (`curl -fsSL https://bun.sh/install | bash`)
- A **terminal emulator** (the default macOS Terminal works, but Ghostty, WezTerm, Kitty, or Alacritty will give a better experience)
- An **internet connection** (for the AI model API)

---

## Part 1: Install and Set Up OpenCode

### Exercise 1: Install OpenCode

1. Go to [https://opencode.ai](https://opencode.ai) and follow the installation instructions for your platform.

2. Verify the installation:
   ```bash
   opencode --version
   ```

### Exercise 2: Select a Free Model

1. Start OpenCode (from any directory for now):
   ```bash
   opencode
   ```

2. Type `/models` and select one of the free models. No login or API key is required — free models work out of the box.

3. Type `/exit` to quit for now. We'll start a real session in Part 3.

---

## Part 2: Choose Your Project

This is a **freestyle mini hackathon**. You'll build something of your own choosing using OpenCode as your coding partner. The only rules:

- **Use Bun and TypeScript** (so you can apply what you've learned)
- **Build something that works** — it doesn't need to be polished, but it should run
- **You are the pilot** — direct the agent, review its output, and don't accept code you don't understand

### Exercise 3: Pick an Idea

Choose a project that interests you. If you need inspiration, here are some starting points:

| Category | Ideas |
|---|---|
| **Developer tools** | A git helper CLI, a project scaffolder, a dependency checker, a code snippet manager |
| **Automate the boring stuff** | A file organizer, a bulk renamer, a data format converter, a markdown report generator |
| **Campus life** | A study timer, a grade calculator, an assignment tracker, a schedule planner |
| **API mashup** | A weather dashboard, a trivia quiz (using a free API), a random recipe finder, a Pokémon lookup |
| **Terminal games** | Snake, tic-tac-toe, hangman, a quiz game, a text adventure |
| **Solve a daily annoyance** | Something that fixes a repetitive task or slow workflow in your day |

You can also come up with something completely different. The point is to experience the agentic loop — prompt, review, refine — on a project you actually care about.

### Exercise 4: Set Up Your Project

1. Create a project directory and initialize it:
   ```bash
   mkdir week-12-hackathon && cd week-12-hackathon
   bun init -y
   ```

2. Create a basic directory structure:
   ```bash
   mkdir -p src
   ```

3. Start OpenCode in your project:
   ```bash
   opencode
   ```

4. Set context with your first prompt. Describe what you want to build. For example:

   ```
   I want to build a CLI tool with Bun and TypeScript that [describe your idea].
   It should [list the main features]. Let's start by setting up the project
   structure and implementing the core functionality.
   ```

   The more specific you are, the better the agent's output will be.

---

## Part 3: Build

### Exercise 5: Work Through the Agentic Loop

Build your project by working through iterations with the agent. Here's a suggested flow:

1. **Start with the core** — Ask the agent to implement the most basic version of your idea first. Get something working before adding features.

2. **Test as you go** — After each chunk of code the agent writes, run it. Does it work? If not, tell the agent what's broken.

3. **Add features incrementally** — Once the core works, add features one at a time. Each prompt should be a small, clear request.

4. **Review the code** — Don't just check if it runs. Read the code. Does it make sense? Is it organized well? Would you write it differently?

5. **Push back when needed** — If the agent makes a choice you disagree with or produces something you don't understand, say so. Ask it to explain or change its approach.

**Tips for good prompts:**

| Tip | Example |
|---|---|
| **Set context first** | "Look at the project. I'm building X with Y." |
| **Be specific** | "Create a function that takes X and returns Y" not "make it work" |
| **Describe constraints** | "Use bun:test, not Jest. Store data as JSON, not SQLite." |
| **Review before continuing** | Read the generated code before asking for the next feature |
| **Point out mistakes** | "The search function isn't case-insensitive. Fix that." |
| **Ask for explanations** | "Why did you use X instead of Y?" |

### Exercise 6: Add Tests

Once you have something working, ask the agent to write tests:

```
Write tests for [your core functionality] using bun:test.
Make sure they cover both the happy path and edge cases.
```

Run them:

```bash
bun test
```

Review the tests the agent wrote. Are they meaningful? Do they actually test behavior, or are they just checking that functions exist? Ask the agent to improve them if needed.

---

## Part 4: Reflect

### Exercise 7: Review What You Built

This is the most important part of the exercise. Look at the code the agent generated and think about these questions (discuss with a classmate or write down notes):

1. **Code quality** — Is the code well-structured? Would you organize it differently? Are there any obvious bugs or edge cases the agent missed?

2. **Testing** — Are the tests meaningful? Do they actually catch bugs, or are they just "happy path" tests? What test cases are missing?

3. **What you caught** — During the exercise, did you catch any mistakes in the agent's output? What knowledge helped you spot them?

4. **What you guided** — How did your prompts shape the output? Compare the first prompt you wrote to the last. Did you get better at directing the agent?

5. **The 11-week connection** — How would your experience with this exercise be different if you hadn't taken this course? Could you have evaluated the agent's output on testing, error handling, or code structure without the knowledge from earlier weeks?

---

## Quick Reference

### OpenCode Commands

| Command | What It Does |
|---|---|
| `/models` | Select which model to use |
| `/connect` | Add or configure an AI provider |
| `/init` | Analyze project and create AGENTS.md |
| `/new` | Start a new conversation session |
| `/undo` | Undo the last change the agent made |
| `/exit` | Exit OpenCode |
| `Tab` | Toggle between Plan mode (read-only) and Build mode |
| `@` | Search for a file to reference in your prompt |

### The Agentic Engineering Mindset

| Principle | What It Means |
|---|---|
| **You are the pilot** | The agent executes, but you set direction and evaluate quality |
| **Knowledge is leverage** | The more you know, the better you can guide and review |
| **Review everything** | AI-generated code is a first draft, not a final product |
| **Iterate quickly** | Short prompt → review → refine cycles beat long detailed prompts |
| **Trust but verify** | The agent is confident even when wrong — always check |

---

## Next Steps

You've now experienced the full loop of modern software development:

| Week | What You Learned | How It Helps with AI Agents |
|---|---|---|
| 1-4 | TypeScript, HTTP servers, Bun | You can read and evaluate generated code |
| 5 | Testing | You know if generated tests are actually meaningful |
| 6 | CI/CD with GitHub Actions | You can verify the agent sets up pipelines correctly |
| 7 | Docker | You can evaluate generated Dockerfiles and compose files |
| 8 | Databases with Drizzle | You can review schema design and query logic |
| 9 | Kubernetes | You can check generated manifests and deployments |
| 10 | Terraform | You can validate infrastructure-as-code configurations |
| 11 | Monitoring with Datadog | You can ensure proper logging and metrics instrumentation |
| **12** | **Agentic engineering** | **You can use all of the above to guide AI effectively** |

The tools will keep getting better. Your ability to evaluate, guide, and guard their output is what makes you an engineer — not just a prompt writer.

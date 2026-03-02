# Week 8: Managing Data with Drizzle ORM

## Objective

Learn how to manage application data using **Drizzle ORM** with SQLite and Bun. By the end, you'll understand database schemas, migrations, CRUD operations, seed data, test isolation, and CI integration — skills that apply to any real-world project.

---

## Why Data Management Matters

Code is disposable — you can always rewrite it. **Data is not.** In real-world systems:

- Data outlives the application that created it
- Data must be preserved across deployments and rollbacks
- Data grows and changes over time — change is inevitable
- Every environment (development, staging, production) needs its own managed database

This means you can't manage data by hand. You need **automation, scripting, and version control** for your database — just like your code.

---

## Database Scripting and Migrations

### Why Script Your Database?

| Manual Database Changes | Scripted Database Changes |
|---|---|
| "I ran this SQL on production last Tuesday" | Every change is a versioned file in Git |
| Different structure per environment | Same script runs on all environments |
| Can't reproduce a database from scratch | `bun run db:migrate` and you're done |
| Rollbacks are guesswork | Rollbacks are just another migration |

### How Migrations Work

A **migration** is a script that changes your database structure. Migrations are numbered and run in order:

```
migrations/
├── 0000_create_users_table.sql    ← First migration (UP)
├── 0001_add_age_to_users.sql      ← Second migration (UP)
└── 0002_create_posts_table.sql    ← Third migration (UP)
```

Each migration moves the database **forward** one version. The migration tool tracks which migrations have already been applied, so it only runs new ones.

### Drizzle's Approach

Drizzle ORM takes a **"codebase first"** approach:

1. You define your schema in TypeScript
2. Drizzle compares your schema to the current database
3. It generates SQL migration files automatically
4. You apply those migrations to the database

This means your schema is always defined in code, version-controlled, and type-safe.

---

## Why Drizzle ORM?

| Feature | Raw SQL | Prisma | Drizzle |
|---|---|---|---|
| Type safety | None | Generated types | Schema IS the types |
| SQL knowledge needed | Yes | No (abstracted away) | Yes (SQL-like API) |
| Migration approach | Manual files | Auto-generated | Auto-generated |
| Bundle size | N/A | Heavy | Lightweight |
| Learning curve | Low (if you know SQL) | Medium | Low (if you know SQL) |
| Query syntax | SQL strings | Custom DSL | Looks like SQL |

Drizzle's API mirrors SQL closely — if you know SQL, you already know Drizzle. And because your schema is TypeScript, you get full autocompletion and type checking.

---

## Prerequisites

- **Bun** installed (https://bun.sh/)
- Basic knowledge of TypeScript

---

## Project Structure (What You'll Build)

```
week-8-data/
├── src/
│   └── db/
│       ├── index.ts       ← Database connection
│       ├── schema.ts      ← Table definitions
│       ├── migrate.ts     ← Migration runner
│       └── seed.ts        ← Seed data script
├── tests/
│   └── posts.test.ts      ← Database tests
├── drizzle/                ← Generated migration SQL files
│   └── 0000_*.sql
├── .github/
│   └── workflows/
│       └── tests.yml      ← CI workflow
├── drizzle.config.ts       ← Drizzle Kit configuration
├── package.json
└── tsconfig.json
```

---

## Part 1: Set Up the Project

### Exercise 1: Create a Bun Project

1. Create and enter the project directory:
   ```bash
   mkdir week-8-data && cd week-8-data
   ```

2. Initialize a Bun project:
   ```bash
   bun init -y
   ```

   This creates `package.json`, `tsconfig.json`, and an `index.ts` file.

### Exercise 2: Install Drizzle ORM

1. Install Drizzle ORM and Drizzle Kit:
   ```bash
   bun add drizzle-orm
   bun add -d drizzle-kit better-sqlite3
   ```

   **What each package does:**
   - `drizzle-orm` — The ORM itself: schema definitions, query builder, type-safe queries
   - `drizzle-kit` — CLI tool for generating migrations and running Drizzle Studio
   - `better-sqlite3` — SQLite driver required by Drizzle Kit's tooling (Studio). Your application code uses Bun's built-in `bun:sqlite` instead — `better-sqlite3` is only a dev dependency.

   **What about the database driver?** Bun includes a built-in SQLite driver (`bun:sqlite`) — no extra runtime package needed. We'll use `drizzle-orm/bun-sqlite` to connect Drizzle to Bun's SQLite. The `better-sqlite3` package is only needed so Drizzle Kit's CLI tools (like Studio) can connect to the database.

2. Create the Drizzle configuration file `drizzle.config.ts` in the project root:

   ```typescript
   import { defineConfig } from "drizzle-kit";

   export default defineConfig({
     dialect: "sqlite",
     schema: "./src/db/schema.ts",
     out: "./drizzle",
     dbCredentials: {
       url: "./sqlite.db",
     },
   });
   ```

   **What this does:**
   - `dialect` — Which database engine to use (SQLite in our case)
   - `schema` — Path to your TypeScript schema file
   - `out` — Where to write generated migration SQL files
   - `dbCredentials.url` — Path to the SQLite database file (used by Drizzle Studio)

3. Add `*.db` and related files to `.gitignore`:
   ```
   *.db
   *.db-shm
   *.db-wal
   ```

---

## Part 2: Define Your Schema

### Exercise 3: Create the Database Schema

A schema defines the structure of your database — what tables exist and what columns they have. In Drizzle, you define this in TypeScript.

1. Create the database directory:
   ```bash
   mkdir -p src/db
   ```

2. Create `src/db/schema.ts`:

   ```typescript
   import { sqliteTable, integer, text } from "drizzle-orm/sqlite-core";
   import { sql } from "drizzle-orm";

   export const posts = sqliteTable("posts", {
     id: integer("id").primaryKey({ autoIncrement: true }),
     title: text("title").notNull(),
     content: text("content").notNull(),
     published: integer("published", { mode: "boolean" }).notNull().default(false),
     createdAt: text("created_at")
       .notNull()
       .default(sql`(CURRENT_TIMESTAMP)`),
   });
   ```

   **How this maps to SQL:**

   | Drizzle TypeScript | SQL Equivalent |
   |---|---|
   | `sqliteTable("posts", {...})` | `CREATE TABLE posts (...)` |
   | `integer("id").primaryKey({ autoIncrement: true })` | `id INTEGER PRIMARY KEY AUTOINCREMENT` |
   | `text("title").notNull()` | `title TEXT NOT NULL` |
   | `integer("published", { mode: "boolean" })` | `published INTEGER` (stored as 0/1) |
   | `.default(sql\`(CURRENT_TIMESTAMP)\`)` | `DEFAULT (CURRENT_TIMESTAMP)` |

   **Why define schema in TypeScript?** Your database structure lives in version control alongside your code. When you query the database, TypeScript knows the exact shape of your data — you get autocompletion and type checking for free.

### Exercise 4: Generate and Apply Your First Migration

Now let's turn your TypeScript schema into a real database.

1. Generate the migration:
   ```bash
   bunx drizzle-kit generate
   ```

   This compares your schema to the database (which doesn't exist yet) and creates a SQL migration file in the `drizzle/` folder.

2. Look at the generated file:
   ```bash
   ls drizzle/
   ```

   You'll see a file like `0000_create_posts_table.sql`. Open it — it's standard SQL:

   ```sql
   CREATE TABLE `posts` (
     `id` integer PRIMARY KEY AUTOINCREMENT NOT NULL,
     `title` text NOT NULL,
     `content` text NOT NULL,
     `published` integer DEFAULT false NOT NULL,
     `created_at` text DEFAULT (CURRENT_TIMESTAMP) NOT NULL
   );
   ```

   **This is the migration.** It's a versioned SQL file that Drizzle generated from your TypeScript schema. It goes into Git just like your code.

3. Now we need to apply the migration. Rather than using Drizzle Kit's built-in `migrate` command (which uses `better-sqlite3` under the hood), we'll write a small migration runner that uses Bun's built-in `bun:sqlite`. This keeps our application code independent of Node.js-specific drivers.

   Create `src/db/migrate.ts`:

   ```typescript
   import { Database } from "bun:sqlite";
   import { readdir, readFile } from "fs/promises";
   import { join } from "path";

   const DB_PATH = "sqlite.db";
   const MIGRATIONS_DIR = join(import.meta.dir, "../../drizzle");

   const db = new Database(DB_PATH);
   db.run("PRAGMA journal_mode = WAL");

   // Create migrations tracking table
   db.run(`
     CREATE TABLE IF NOT EXISTS __drizzle_migrations (
       id INTEGER PRIMARY KEY AUTOINCREMENT,
       hash TEXT NOT NULL,
       created_at INTEGER NOT NULL
     )
   `);

   // Get already applied migrations
   const applied = db
     .query("SELECT hash FROM __drizzle_migrations")
     .all()
     .map((row: any) => row.hash);

   // Read and sort migration files
   const files = (await readdir(MIGRATIONS_DIR))
     .filter((f) => f.endsWith(".sql"))
     .sort();

   console.log(`Found ${files.length} migration(s), ${applied.length} already applied`);

   let newMigrations = 0;
   for (const file of files) {
     const hash = file.replace(".sql", "");
     if (applied.includes(hash)) {
       continue;
     }

     const sql = await readFile(join(MIGRATIONS_DIR, file), "utf-8");

     // Split by Drizzle's statement breakpoint marker
     const statements = sql
       .split("--> statement-breakpoint")
       .map((s) => s.trim())
       .filter(Boolean);

     db.transaction(() => {
       for (const stmt of statements) {
         db.run(stmt);
       }
       db.run(
         "INSERT INTO __drizzle_migrations (hash, created_at) VALUES (?, ?)",
         [hash, Date.now()]
       );
     })();

     console.log(`✓ Applied: ${file}`);
     newMigrations++;
   }

   if (newMigrations === 0) {
     console.log("No new migrations to apply.");
   } else {
     console.log(`Applied ${newMigrations} new migration(s).`);
   }

   db.close();
   ```

   **What this does:**
   - Uses Bun's built-in `bun:sqlite` to read the generated SQL files and execute them
   - Tracks which migrations have already been applied (just like any migration tool)
   - Runs each migration inside a transaction for safety
   - Is idempotent — running it multiple times is safe

4. Run the migration:
   ```bash
   bun src/db/migrate.ts
   ```

   You should see output confirming the migration was applied and the `posts` table was created.

5. Verify with Drizzle Studio:
   ```bash
   bunx drizzle-kit studio
   ```

   Open the URL shown in the terminal (usually https://local.drizzle.studio). You'll see your `posts` table with its columns. The table is empty — we'll add data next. Stop Studio with `Ctrl+C`.

6. Add scripts to `package.json` for convenience:
   ```json
   {
     "scripts": {
       "db:generate": "drizzle-kit generate",
       "db:migrate": "bun src/db/migrate.ts",
       "db:studio": "drizzle-kit studio"
     }
   }
   ```

   Now you can run `bun run db:generate`, `bun run db:migrate`, and `bun run db:studio`.

---

## Part 3: Connect and Query

### Exercise 5: Set Up the Database Connection

1. Create `src/db/index.ts`:

   ```typescript
   import { drizzle } from "drizzle-orm/bun-sqlite";
   import { Database } from "bun:sqlite";
   import * as schema from "./schema";

   const sqlite = new Database("sqlite.db");

   export const db = drizzle(sqlite, { schema });
   ```

   **What this does:**
   - Uses Bun's built-in SQLite driver (`bun:sqlite`) — no extra packages needed
   - Passes the database instance and your schema to Drizzle
   - Exports `db` so you can import it anywhere in your project

### Exercise 6: Seed the Database

Seed scripts populate your database with initial data. This is essential for:
- Setting up development environments quickly
- Having consistent test data
- Reproducing bugs with known data

1. Create `src/db/seed.ts`:

   ```typescript
   import { db } from "./index";
   import { posts } from "./schema";

   const seedPosts = [
     {
       title: "Getting Started with Drizzle ORM",
       content:
         "Drizzle ORM is a lightweight TypeScript ORM that lets you write database queries that look like SQL.",
       published: true,
     },
     {
       title: "Understanding Database Migrations",
       content:
         "Migrations are versioned scripts that change your database structure. They ensure every environment has the same schema.",
       published: true,
     },
     {
       title: "Draft: Advanced Query Patterns",
       content:
         "This post covers joins, aggregations, and subqueries in Drizzle.",
       published: false,
     },
   ];

   async function seed() {
     console.log("Seeding database...");

     // Clear existing data
     await db.delete(posts);

     // Insert seed data
     const inserted = await db.insert(posts).values(seedPosts).returning();

     console.log(`Inserted ${inserted.length} posts:`);
     for (const post of inserted) {
       console.log(`  - ${post.title} (${post.published ? "published" : "draft"})`);
     }

     console.log("Seeding complete.");
   }

   seed();
   ```

   **Key concepts:**
   - `db.delete(posts)` — Deletes all rows from the posts table (clean slate)
   - `db.insert(posts).values([...])` — Inserts multiple rows at once
   - `.returning()` — Returns the inserted rows (including auto-generated `id` and `createdAt`)
   - The seed script is **idempotent** — you can run it multiple times safely because it clears old data first

2. Add a seed script to `package.json`:
   ```json
   {
     "scripts": {
       "db:seed": "bun src/db/seed.ts"
     }
   }
   ```

3. Run the seed:
   ```bash
   bun run db:seed
   ```

   You should see output showing 3 posts were inserted.

4. Verify in Drizzle Studio:
   ```bash
   bun run db:studio
   ```

   Open the Studio URL and you should see your 3 posts in the table.

### Exercise 7: Query Data from the Database

Let's practice Drizzle's query API by writing a small script.

1. Create `src/db/playground.ts`:

   ```typescript
   import { db } from "./index";
   import { posts } from "./schema";
   import { eq } from "drizzle-orm";

   // Select all posts
   console.log("--- All posts ---");
   const allPosts = await db.select().from(posts);
   console.log(allPosts);

   // Select only published posts
   console.log("\n--- Published posts ---");
   const published = await db
     .select()
     .from(posts)
     .where(eq(posts.published, true));
   console.log(published);

   // Select specific columns
   console.log("\n--- Titles only ---");
   const titles = await db
     .select({ id: posts.id, title: posts.title })
     .from(posts);
   console.log(titles);

   // Update a post
   console.log("\n--- Publishing draft post ---");
   const updated = await db
     .update(posts)
     .set({ published: true })
     .where(eq(posts.published, false))
     .returning();
   console.log("Updated:", updated);

   // Delete a post (by id of the last one)
   const last = allPosts[allPosts.length - 1];
   console.log(`\n--- Deleting post ${last.id} ---`);
   const deleted = await db
     .delete(posts)
     .where(eq(posts.id, last.id))
     .returning();
   console.log("Deleted:", deleted);

   // Final state
   console.log("\n--- Final state ---");
   const remaining = await db.select().from(posts);
   console.log(remaining);
   ```

2. Run the playground:
   ```bash
   bun src/db/playground.ts
   ```

   Observe how each query returns typed data. The Drizzle query syntax mirrors SQL closely:

   | Drizzle | SQL |
   |---|---|
   | `db.select().from(posts)` | `SELECT * FROM posts` |
   | `db.select().from(posts).where(eq(posts.published, true))` | `SELECT * FROM posts WHERE published = 1` |
   | `db.select({ id: posts.id, title: posts.title }).from(posts)` | `SELECT id, title FROM posts` |
   | `db.update(posts).set({ published: true }).where(...)` | `UPDATE posts SET published = 1 WHERE ...` |
   | `db.delete(posts).where(eq(posts.id, 3))` | `DELETE FROM posts WHERE id = 3` |

3. Re-seed the database to restore the original data:
   ```bash
   bun run db:seed
   ```

4. You can delete `src/db/playground.ts` — it was just for learning. Or keep it for experimentation.

---

## Part 4: Schema Changes and Migrations

This is where data management gets real. Your schema will change over time — new columns, new tables, renamed fields. Migrations handle this safely.

### Exercise 8: Add a New Column (Incremental Change)

Let's add an `author` column to the posts table.

1. Update `src/db/schema.ts` — add the `author` field:

   ```typescript
   import { sqliteTable, integer, text } from "drizzle-orm/sqlite-core";
   import { sql } from "drizzle-orm";

   export const posts = sqliteTable("posts", {
     id: integer("id").primaryKey({ autoIncrement: true }),
     title: text("title").notNull(),
     content: text("content").notNull(),
     author: text("author").notNull().default("Anonymous"),
     published: integer("published", { mode: "boolean" }).notNull().default(false),
     createdAt: text("created_at")
       .notNull()
       .default(sql`(CURRENT_TIMESTAMP)`),
   });
   ```

   **Why `.default("Anonymous")`?** Existing rows in the database don't have an `author` value. The default ensures the migration doesn't fail — existing posts get "Anonymous" as their author.

2. Generate the migration:
   ```bash
   bun run db:generate
   ```

   Look at the new file in `drizzle/`. It should contain something like:

   ```sql
   ALTER TABLE `posts` ADD `author` text DEFAULT 'Anonymous' NOT NULL;
   ```

   This is an **incremental change** — it modifies the existing table without destroying data.

3. Apply the migration:
   ```bash
   bun run db:migrate
   ```

4. Verify in Drizzle Studio:
   ```bash
   bun run db:studio
   ```

   Your existing posts now have the `author` column set to "Anonymous".

### Exercise 9: Add a New Table with a Relationship

Let's add a `comments` table that references `posts`.

1. Update `src/db/schema.ts` — add the comments table below the posts table:

   ```typescript
   export const comments = sqliteTable("comments", {
     id: integer("id").primaryKey({ autoIncrement: true }),
     postId: integer("post_id")
       .notNull()
       .references(() => posts.id),
     author: text("author").notNull(),
     body: text("body").notNull(),
     createdAt: text("created_at")
       .notNull()
       .default(sql`(CURRENT_TIMESTAMP)`),
   });
   ```

   **Key concept:** `.references(() => posts.id)` creates a **foreign key** — every comment must belong to an existing post. The database will reject any comment with an invalid `post_id`.

2. Generate and apply the migration:
   ```bash
   bun run db:generate && bun run db:migrate
   ```

3. Update the seed script `src/db/seed.ts` to include comments:

   ```typescript
   import { db } from "./index";
   import { posts, comments } from "./schema";

   const seedPosts = [
     {
       title: "Getting Started with Drizzle ORM",
       content:
         "Drizzle ORM is a lightweight TypeScript ORM that lets you write database queries that look like SQL.",
       author: "Alice",
       published: true,
     },
     {
       title: "Understanding Database Migrations",
       content:
         "Migrations are versioned scripts that change your database structure. They ensure every environment has the same schema.",
       author: "Bob",
       published: true,
     },
     {
       title: "Draft: Advanced Query Patterns",
       content:
         "This post covers joins, aggregations, and subqueries in Drizzle.",
       author: "Alice",
       published: false,
     },
   ];

   async function seed() {
     console.log("Seeding database...");

     // Clear existing data (comments first due to foreign key)
     await db.delete(comments);
     await db.delete(posts);

     // Insert seed data and get back the inserted rows (with their IDs)
     const insertedPosts = await db.insert(posts).values(seedPosts).returning();
     console.log(`Inserted ${insertedPosts.length} posts`);

     // Use actual post IDs for comments (IDs auto-increment, so we can't hardcode them)
     const seedComments = [
       { postId: insertedPosts[0].id, author: "Bob", body: "Great introduction! Very clear." },
       { postId: insertedPosts[0].id, author: "Charlie", body: "This helped me get started quickly." },
       { postId: insertedPosts[1].id, author: "Alice", body: "Migrations are so important for team work." },
     ];

     const insertedComments = await db.insert(comments).values(seedComments).returning();
     console.log(`Inserted ${insertedComments.length} comments`);

     console.log("Seeding complete.");
   }

   seed();
   ```

   **Why use `insertedPosts[0].id` instead of hardcoding `1`?** SQLite's autoincrement IDs don't reset when you delete rows. After deleting and re-inserting, post IDs might be 4, 5, 6 instead of 1, 2, 3. Using the returned IDs ensures comments always reference existing posts.

4. Run the seed:
   ```bash
   bun run db:seed
   ```

5. Verify in Drizzle Studio that both tables have data:
   ```bash
   bun run db:studio
   ```

---

## Part 5: Test Data and Isolation

When testing applications that use a database, you need a strategy for managing test data. The slides cover three approaches:

| Approach | How It Works | Tradeoff |
|---|---|---|
| **Test isolation** (Recommended) | Each test gets a fresh database state | Slowest but most reliable |
| **Adaptive tests** | Tests check current state and adapt | Complex test logic |
| **Test sequencing** | Tests run in a specific order, depending on previous tests | Fragile — one failure breaks everything |

**Test isolation** is the recommended approach: return the database to a known state before each test.

### Exercise 10: Add a Database Reset Script

1. Add a reset script to `package.json` that drops and recreates everything:
   ```json
   {
     "scripts": {
       "db:reset": "rm -f sqlite.db && bun src/db/migrate.ts && bun src/db/seed.ts"
     }
   }
   ```

2. Run it:
   ```bash
   bun run db:reset
   ```

   This gives you a completely fresh database with known seed data — useful for:
   - Starting over during development
   - Setting up CI test environments
   - Reproducing bugs with consistent data

---

## Part 6: Write Database Tests

Bun includes a built-in test runner (`bun:test`) — no extra test framework needed. The API is similar to Jest and Vitest.

### Exercise 11: Set Up Testing

1. Create the test directory:
   ```bash
   mkdir tests
   ```

2. Add test scripts to `package.json`:
   ```json
   {
     "scripts": {
       "test": "bun test",
       "test:watch": "bun test --watch"
     }
   }
   ```

### Exercise 12: Write Database Tests

These tests operate directly on the database. Each test resets the data first to ensure a known state.

1. Create `tests/posts.test.ts`:

   ```typescript
   import { describe, it, expect, beforeEach } from "bun:test";
   import { db } from "../src/db/index";
   import { posts, comments } from "../src/db/schema";
   import { eq } from "drizzle-orm";

   const seedData = [
     { title: "First Post", content: "Content 1", author: "Alice", published: true },
     { title: "Second Post", content: "Content 2", author: "Bob", published: false },
   ];

   // Reset database before EACH test — test isolation
   beforeEach(async () => {
     await db.delete(comments);
     await db.delete(posts);
     await db.insert(posts).values(seedData);
   });

   describe("Posts CRUD", () => {
     it("should list all posts", async () => {
       const result = await db.select().from(posts);
       expect(result).toHaveLength(2);
     });

     it("should create a new post", async () => {
       const [newPost] = await db
         .insert(posts)
         .values({ title: "New Post", content: "New content", author: "Charlie" })
         .returning();

       expect(newPost.title).toBe("New Post");
       expect(newPost.id).toBeDefined();
       expect(newPost.published).toBe(false); // default value

       const all = await db.select().from(posts);
       expect(all).toHaveLength(3);
     });

     it("should filter published posts", async () => {
       const published = await db
         .select()
         .from(posts)
         .where(eq(posts.published, true));

       expect(published).toHaveLength(1);
       expect(published[0].title).toBe("First Post");
     });

     it("should update a post", async () => {
       const allPosts = await db.select().from(posts);
       const firstPost = allPosts[0];

       const [updated] = await db
         .update(posts)
         .set({ title: "Updated Title" })
         .where(eq(posts.id, firstPost.id))
         .returning();

       expect(updated.title).toBe("Updated Title");
       expect(updated.content).toBe("Content 1"); // unchanged
     });

     it("should delete a post", async () => {
       const allPosts = await db.select().from(posts);
       const firstPost = allPosts[0];

       const [deleted] = await db
         .delete(posts)
         .where(eq(posts.id, firstPost.id))
         .returning();

       expect(deleted.title).toBe("First Post");

       const remaining = await db.select().from(posts);
       expect(remaining).toHaveLength(1);
     });

     it("should not affect other tests (isolation check)", async () => {
       // This test runs after the delete test, but should still see 2 posts
       // because beforeEach resets the database
       const result = await db.select().from(posts);
       expect(result).toHaveLength(2);
     });
   });
   ```

   **Key pattern:** The `beforeEach` hook deletes all data and re-inserts the seed data before every test. This means:
   - Tests can insert, update, and delete freely without worrying about cleanup
   - Tests never depend on each other's state
   - Tests can run in any order
   - The last test ("isolation check") proves this works

2. Run the tests:
   ```bash
   bun test
   ```

   All 6 tests should pass. The isolation test proves that the delete from the previous test didn't leak into this one.

---

## Part 7: CI Integration

### Exercise 13: Add Database to CI (GitHub Actions)

When running tests in CI, you need to create the database from scratch. This is where your migration and seed scripts pay off.

1. Create `.github/workflows/tests.yml`:

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

         - name: Run migrations
           run: bun run db:migrate

         - name: Seed database
           run: bun run db:seed

         - name: Run tests
           run: bun test
   ```

   **Notice:** The CI pipeline uses the exact same scripts you use locally. No special CI database setup — just `migrate` and `seed`.

---

## Challenge Exercises

### Challenge 1: Add Comment Tests

Write tests for the comments table in `tests/comments.test.ts`:
1. Test that inserting a comment with a valid `postId` works
2. Test that inserting a comment with an invalid `postId` fails (foreign key violation)
3. Test that deleting a post also considers its comments (what happens?)

### Challenge 2: Add Pagination Helpers

1. Create a `src/db/queries.ts` file with reusable query functions:
   ```typescript
   export async function getPaginatedPosts(page: number, limit: number) { ... }
   ```
2. Use Drizzle's `.limit()` and `.offset()` methods
3. Return both the results and the total count
4. Write tests that verify pagination works correctly

### Challenge 3: Add a Search Function

1. Add a search function to `src/db/queries.ts`:
   ```typescript
   import { like, or } from "drizzle-orm";

   export async function searchPosts(query: string) {
     return db
       .select()
       .from(posts)
       .where(
         or(
           like(posts.title, `%${query}%`),
           like(posts.content, `%${query}%`)
         )
       );
   }
   ```
2. Write tests that verify search returns correct results

---

## Quick Reference

### Drizzle Schema — SQLite Column Types

| Drizzle | SQLite Type | TypeScript Type |
|---|---|---|
| `integer("col")` | INTEGER | `number` |
| `integer("col", { mode: "boolean" })` | INTEGER (0/1) | `boolean` |
| `text("col")` | TEXT | `string` |
| `real("col")` | REAL | `number` |
| `.primaryKey({ autoIncrement: true })` | PRIMARY KEY AUTOINCREMENT | `number` |
| `.notNull()` | NOT NULL | removes `null` from type |
| `.default(value)` | DEFAULT value | — |
| `.references(() => table.col)` | FOREIGN KEY | — |

### Drizzle Query API

| Operation | Code |
|---|---|
| Select all | `db.select().from(table)` |
| Select with filter | `db.select().from(table).where(eq(table.col, value))` |
| Select columns | `db.select({ col1: table.col1 }).from(table)` |
| Insert one | `db.insert(table).values({ ... }).returning()` |
| Insert many | `db.insert(table).values([{ ... }, { ... }]).returning()` |
| Update | `db.update(table).set({ col: value }).where(...).returning()` |
| Delete | `db.delete(table).where(...).returning()` |
| Count | `db.select({ count: count() }).from(table)` |

### Scripts

| Command | Description |
|---|---|
| `bun run db:generate` | Generate SQL migration from schema changes |
| `bun run db:migrate` | Apply pending migrations to the database |
| `bun run db:seed` | Populate database with seed data |
| `bun run db:reset` | Drop database, re-migrate, and re-seed |
| `bun run db:studio` | Open Drizzle Studio (visual database browser) |

### Filter Operators

```typescript
import { eq, ne, lt, gt, lte, gte, like, and, or, not } from "drizzle-orm";

eq(col, value)     // col = value
ne(col, value)     // col != value
lt(col, value)     // col < value
gt(col, value)     // col > value
like(col, "%val%") // col LIKE '%val%'
and(cond1, cond2)  // cond1 AND cond2
or(cond1, cond2)   // cond1 OR cond2
```

---

## Next Steps

After completing these exercises, explore:
- **Drizzle Relations** — Define relationships between tables for nested queries
- **Transactions** — Wrap multiple operations in a single atomic unit
- **Drizzle with PostgreSQL** — Use the same API with a production database
- **Database indexes** — Speed up queries on large tables
- **Using Drizzle in a web framework** — Connect your database to Next.js, Hono, or Elysia API routes

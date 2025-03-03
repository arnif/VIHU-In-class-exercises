Adding Prisma to a Node.js project involves several steps, from setting up Prisma itself to integrating it with your Node.js application. Here's a step-by-step guide to help you through the process:

Below is an updated guide that uses Bun instead of npm:

---

### 1. Setup Your Bun Project

If you haven't already, create a new Bun project:

```bash
mkdir my-bun-project
cd my-bun-project
bun init
```

This initializes your project and creates a `package.json` for you.

---

### 2. Install Prisma CLI

Install Prisma CLI as a development dependency using Bun:

```bash
bun add prisma --dev
```

---

### 3. Initialize Prisma in Your Project

Run the Prisma initialization command using Bun’s equivalent of npx (bunx):

```bash
bunx prisma init
```

This command creates a `prisma` folder with a `schema.prisma` file and a `.env` file in your project's root directory.

---

### 4. Configure Your Database Connection for SQLite

In the `.env` file, configure your database connection to use SQLite. For example:

```
DATABASE_URL="file:./dev.db"
```

This tells Prisma to use a local SQLite database file named `dev.db` in your project’s root.

---

### 5. Define Your Data Model

Open the `schema.prisma` file in the `prisma` directory and define your data model. For instance:

```prisma
model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
}

model User {
  id    Int     @id @default(autoincrement())
  name  String
  email String  @unique
  posts Post[]
}
```

---

### 6. Generate Prisma Client

Generate the Prisma Client with the following command:

```bash
bunx prisma generate
```

This step creates the Prisma Client based on your defined data model.

---

### 7. Use Prisma Client in Your Bun Application

First, install the Prisma Client package:

```bash
bun add @prisma/client
```

Then, integrate it into your application. For example, create an `index.js` file with the following code:

```javascript
const { PrismaClient } = require('@prisma/client');

const prisma = new PrismaClient();

async function main() {
  const newUser = await prisma.user.create({
    data: {
      name: 'Alice',
      email: 'alice@example.com',
    },
  });
  console.log('Created new user:', newUser);
}

main()
  .catch((e) => {
    throw e;
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

---

### 8. Run Your Bun Application

Execute your application using Bun:

```bash
bun index.js
```

---

Below are additional steps (Steps 9–11) that show how to perform a simple migration after running your application:

---

### 9. Modify Your Data Model

To demonstrate a migration, open your `schema.prisma` file and add a new field. For example, you can add an optional `age` field to the `User` model:

```prisma
model User {
  id    Int     @id @default(autoincrement())
  name  String
  email String  @unique
  age   Int?    // new optional field for user age
  posts Post[]
}
```

---

### 10. Create and Apply a Migration

Generate and apply a new migration that reflects the changes to your data model. Run the following command:

```bash
bunx prisma migrate dev --name add-age-to-user
```

This command creates a new migration file (with the name `add-age-to-user`) and applies it to your SQLite database. You should see output confirming that the migration was generated and applied successfully.

---

### 11. Verify the Migration

After the migration completes, you can verify that your changes have been applied by either:
- Inspecting the generated migration files in the `prisma/migrations` directory, or
- Launching Prisma Studio to view your database:

```bash
bunx prisma studio
```

Using Prisma Studio, you can browse the updated schema and verify that the new `age` field appears in the `User` model.

---

By following these additional steps, you'll learn how to modify your Prisma data model and execute a simple migration using Bun.

### Additional Notes for SQLite

- **Database File**: The SQLite database file (`dev.db` in this example) is created automatically in your project's root when you run your application or execute a Prisma command that accesses the database.

By following these steps, you'll have integrated Prisma with a Bun-based Node.js project, enjoying both Bun's fast runtime and Prisma's robust database management with SQLite.
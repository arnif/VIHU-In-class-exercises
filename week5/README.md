# Week 5: Containerizing a Bun App with Docker

## Objective

Learn how to create a small Bun application, run it locally, and then containerize it with Docker. By the end, you'll be able to build and run a container image, troubleshoot common issues, and understand best practices for lightweight containers.

---

## Why Containerize?

| Without Containers | With Containers |
|---|---|
| “Works on my machine” issues | Same environment everywhere |
| Manual setup for new laptops | One command to run |
| Hard to deploy consistently | Image deploys the same way |
| Dependency conflicts | Dependencies isolated per app |

---

## Prerequisites

- **Bun** installed (https://bun.sh/)
- **Docker** installed (https://www.docker.com/products/docker-desktop)
- Basic knowledge of TypeScript/JavaScript

---

## Project Structure (What You’ll Build)

```
my-bun-app/
├── src/
│   └── index.ts
├── package.json
├── bun.lock
├── Dockerfile
├── .dockerignore
└── README.md
```

---

## Exercise 1: Initialize a Bun Project

1. Create a new project folder and initialize Bun:
   ```bash
   mkdir my-bun-app
   cd my-bun-app
   bun init
   ```
2. Confirm Bun runs:
   ```bash
   bun --version
   ```

---

## Exercise 2: Build a Simple HTTP Server

Create `src/index.ts`:

```ts
import { serve } from "bun";

serve({
  port: 3000,
  fetch() {
    return new Response("Hello Bun!", { status: 200 });
  },
});

console.log("Server running at http://localhost:3000");
```

Run it locally:

```bash
bun run src/index.ts
```

Open: `http://localhost:3000`

---

## Exercise 3: Add a Health Check Route

Update `src/index.ts` to support a health endpoint:

```ts
import { serve } from "bun";

serve({
  port: 3000,
  fetch(req) {
    const url = new URL(req.url);
    if (url.pathname === "/health") {
      return new Response(JSON.stringify({ status: "ok" }), {
        headers: { "Content-Type": "application/json" },
      });
    }
    return new Response("Hello Bun!", { status: 200 });
  },
});

console.log("Server running at http://localhost:3000");
```

Test it:

```bash
curl http://localhost:3000/health
```

---

## Exercise 4: Create a Dockerfile

Create `Dockerfile`:

```Dockerfile
# Use the official Bun image as a base image
FROM oven/bun:latest

# Set the working directory
WORKDIR /app

# Copy package.json and lockfile
COPY bun.lock package.json ./

# Install dependencies
RUN bun install

# Copy the rest of the application files
COPY . .

# Expose port 3000
EXPOSE 3000

# Define the command to run your app
CMD ["bun", "run", "src/index.ts"]
```

---

## Exercise 5: Add a .dockerignore

Create `.dockerignore` to keep your image small:

```
node_modules
.git
.gitignore
Dockerfile
README.md
```

---

## Exercise 6: Build and Run the Container

Build the image:

```bash
docker build -t my-bun-app .
```

Run the container:

```bash
docker run -p 3000:3000 my-bun-app
```

Test it:

```bash
curl http://localhost:3000
curl http://localhost:3000/health
```

---

## Exercise 7: Debugging Containers

Check running containers:

```bash
docker ps
```

View logs:

```bash
docker logs <container_id>
```

Stop the container:

```bash
docker stop <container_id>
```

---

## Exercise 8: Environment Variables

Add an environment variable to customize the greeting:

Update `src/index.ts`:

```ts
import { serve } from "bun";

const greeting = process.env.GREETING ?? "Hello Bun!";

serve({
  port: 3000,
  fetch(req) {
    const url = new URL(req.url);
    if (url.pathname === "/health") {
      return new Response(JSON.stringify({ status: "ok" }), {
        headers: { "Content-Type": "application/json" },
      });
    }
    return new Response(greeting, { status: 200 });
  },
});

console.log("Server running at http://localhost:3000");
```

Run with a custom greeting:

```bash
docker run -p 3000:3000 -e GREETING="Hello from Docker!" my-bun-app
```

---

## Exercise 9: Multi-Stage Build (Optional)

You can build smaller images by separating build and runtime stages:

```Dockerfile
# Build stage
FROM oven/bun:latest AS build
WORKDIR /app
COPY bun.lock package.json ./
RUN bun install
COPY . .

# Runtime stage
FROM oven/bun:slim
WORKDIR /app
COPY --from=build /app /app
EXPOSE 3000
CMD ["bun", "run", "src/index.ts"]
```

---

## Exercise 10: Docker Compose (Optional)

Create `docker-compose.yml`:

```yaml
version: "3.9"
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      GREETING: "Hello from Compose!"
```

Run it:

```bash
docker compose up --build
```

---

## Quick Reference

| Command | Description |
|---|---|
| `docker build -t my-bun-app .` | Build image |
| `docker run -p 3000:3000 my-bun-app` | Run container |
| `docker ps` | List running containers |
| `docker logs <id>` | View logs |
| `docker stop <id>` | Stop container |
| `docker compose up --build` | Run with compose |

---

## Next Steps

- Add a database (Postgres, MongoDB) with Docker Compose
- Add tests and run them inside the container
- Deploy the image to a cloud provider (Render, Fly.io, AWS)
- Add GitHub Actions to build/test your image automatically

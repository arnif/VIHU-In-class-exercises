Creating a simple **Bun** application running in a Docker container involves several steps. This guide will take you through the process from setting up a basic Bun application to containerizing it with Docker.

---

## Step 1: Setting Up Your Bun Application

1. **Initialize a new Bun project**: Create a new directory for your project and initialize it with Bun.

   ```bash
   mkdir my-bun-app
   cd my-bun-app
   bun init
   ```

2. **Create your Bun application**: Create a `src` directory, then add an `index.ts` file inside it with the following content:

   ```typescript
   import { serve } from "bun";

   serve({
     port: 3000,
     fetch(req) {
       return new Response("Hello Bun!", { status: 200 });
     },
   });

   console.log("Server running at http://localhost:3000");
   ```

3. **Run your Bun application**:

   ```bash
   bun run src/index.ts
   ```

   This starts your Bun server, which listens on port 3000.

---

## Step 2: Containerizing Your Bun Application with Docker

1. **Install Docker**: If you haven't already, download and install Docker Desktop from [Docker's website](https://www.docker.com/products/docker-desktop) and follow the installation instructions for your operating system.

2. **Create a Dockerfile**: In the root of your project, create a file named `Dockerfile` with the following content:

   ```Dockerfile
   # Use the official Bun image as a base image
   FROM oven/bun:latest

   # Set the working directory
   WORKDIR /app

   # Copy package.json and lockfile
   COPY bun.lockb package.json ./

   # Install dependencies
   RUN bun install

   # Copy the rest of the application files
   COPY . .

   # Expose port 3000
   EXPOSE 3000

   # Define the command to run your app
   CMD ["bun", "run", "src/index.ts"]
   ```

3. **Build your Docker image**: Run the following command in your terminal, replacing `<your-username>/my-bun-app` with your Docker Hub username (if you plan to push the image to Docker Hub) or any name you prefer for local use.

   ```bash
   docker build -t <your-username>/my-bun-app .
   ```

4. **Run your Docker container**: Once the image is built, you can run your container using:

   ```bash
   docker run -p 3000:3000 <your-username>/my-bun-app
   ```

   This command tells Docker to run your container and map port 3000 inside the container to port 3000 on your host, allowing you to access your app at `http://localhost:3000`.

---

## Step 3: Accessing Your Application

- Open a web browser and navigate to `http://localhost:3000`. You should see the "Hello Bun!" message displayed.

---

## Conclusion

Congratulations! You've just containerized a simple Bun application using Docker. This setup is a basic introduction to Dockerizing applications with Bun and can be expanded with more complex configurations, including using Docker Compose for multi-container setups, integrating databases, and deploying to cloud providers.

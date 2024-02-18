Creating a simple Node.js application running in a Docker container involves several steps. This guide will take you through the process from setting up a basic Node.js (TypeScript) application to containerizing it with Docker. I'll assume you're starting with no prior experience with Docker, but have a basic understanding of TypeScript and Node.js.

### Step 1: Setting Up Your Node.js Application

1. **Initialize a new Node.js project**: Create a new directory for your project and initialize it with `npm`.

   ```bash
   mkdir my-node-app
   cd my-node-app
   npm init -y
   ```

2. **Install TypeScript and Node.js types**: Install TypeScript as a development dependency and the types for Node.js.

   ```bash
   npm install --save-dev typescript @types/node
   ```

3. **Create a `tsconfig.json` file**: This file specifies the root files and the compiler options required to compile the project. You can generate a default `tsconfig.json` file by running:

   ```bash
   npx tsc --init
   ```

   Modify the `tsconfig.json` to suit your project. A basic example might look like:

   ```json
   {
     "compilerOptions": {
       "target": "es6",
       "module": "commonjs",
       "rootDir": "./src",
       "outDir": "./dist"
     },
     "include": ["src/**/*"]
   }
   ```

4. **Create your TypeScript application**: Create a `src` directory, then add a `index.ts` file inside it with the following content:

   ```typescript
   import http from "http";

   const hostname = "0.0.0.0";
   const port = 3000;

   const server = http.createServer((req, res) => {
     res.statusCode = 200;
     res.setHeader("Content-Type", "text/plain");
     res.end("Hello World\n");
   });

   server.listen(port, hostname, () => {
     console.log(`Server running at http://${hostname}:${port}/`);
   });
   ```

5. **Compile your TypeScript code**: Add a script to your `package.json` to compile the TypeScript code.

   ```json
   "scripts": {
     "build": "tsc"
   }
   ```

   Run `npm run build` to compile your TypeScript files to JavaScript in the `dist` directory.

### Step 2: Containerizing Your Node.js Application with Docker

1. **Install Docker**: If you haven't already, download and install Docker Desktop from [Docker's website](https://www.docker.com/products/docker-desktop) and follow the installation instructions for your operating system.

2. **Create a Dockerfile**: In the root of your project, create a file named `Dockerfile` with no file extension. This file defines how your Docker container should be built. Here's a simple Dockerfile for your Node.js application:

   ```Dockerfile
   # Use the official Node.js 20 image as a parent image
   FROM node:20

   # Set the working directory in the container
   WORKDIR /usr/src/app

   # Copy package.json and package-lock.json
   COPY package*.json ./

   # Install dependencies
   RUN npm install

   # Bundle your app's source code inside the Docker image
   COPY . .

   # Build your TypeScript app
   RUN npm run build

   # Your app binds to port 3000 so you'll use the EXPOSE instruction to have it mapped by the docker daemon
   EXPOSE 3000

   # Define the command to run your app
   CMD [ "node", "dist/index.js" ]
   ```

3. **Build your Docker image**: Run the following command in your terminal, replacing `<your-username>/my-node-app` with your Docker Hub username (if you plan to push the image to Docker Hub) or any name you prefer for local use.

   ```bash
   docker build -t <your-username>/my-node-app .
   ```

4. **Run your Docker container**: Once the image is built, you can run your container using:

   ```bash
   docker run -p 3000:3000 <your-username>/my-node-app
   ```

   This command tells Docker to run your container and map port 3000 inside the container to port 3000 on your host, allowing you to access your app at `http://localhost:3000`.

### Step 3: Accessing Your Application

- Open a web browser and navigate to `http://localhost:3000`. You should see the "Hello World" message displayed.

### Conclusion

Congratulations! You've just containerized a simple Node.js application using Docker. This setup is a basic introduction to Dockerizing applications and can be expanded with more complex configurations, including using Docker Compose for multi-container setups, integrating databases, and deploying to cloud providers.

## **In-Class Exercise: Logging with Pino in TypeScript using Bun**

### **Objective**
Students will learn how to set up logging in a **Bun + TypeScript** project using **Pino**, explore different log levels, and write logs to a file.

---

## **Step 1: Setup Bun**
1. If you haven’t installed **Bun**, do so first:
   ```sh
   curl -fsSL https://bun.sh/install | bash
   ```
   Restart your terminal if needed.

2. Create a new project:
   ```sh
   mkdir pino-logging-exercise && cd pino-logging-exercise
   ```

3. Initialize a Bun project:
   ```sh
   bun init
   ```
   Choose TypeScript when prompted.

---

## **Step 2: Install Dependencies**
Install **Pino** and **pino-pretty** (for better log readability):
```sh
bun add pino pino-pretty
```

---

## **Step 3: Create a Logger**
Create a new file **logger.ts** and configure **Pino**:
```typescript
import pino from "pino";

// Create a logger instance
const logger = pino({
    level: process.env.LOG_LEVEL || "info", // Use environment variable for log level
    transport: {
        target: "pino-pretty", // Makes logs more readable
        options: { colorize: true },
    },
});

// Export logger
export default logger;
```

---

## **Step 4: Use the Logger**
Create **app.ts** and add the following code:
```typescript
import logger from "./logger";

// Basic logging
logger.info("Application started!");
logger.warn("This is a warning message!");
logger.error("An error occurred!");

// Simulate a function using different log levels
const performTask = (taskName: string) => {
    logger.debug(`Starting task: ${taskName}`);

    if (taskName === "critical-task") {
        logger.error(`Task ${taskName} failed!`);
    } else {
        logger.info(`Task ${taskName} completed successfully.`);
    }
};

// Run tasks
performTask("data-processing");
performTask("critical-task");
```

---

## **Step 5: Run the Code**
Since Bun runs TypeScript natively, there's no need for compilation.

Simply execute:
```sh
bun run app.ts
```

You should see logs in the terminal with different log levels formatted nicely.

---

## **Step 6: Writing Logs to a File**
Modify **logger.ts** to write logs to a file:
```typescript
import pino from "pino";

const logger = pino({
    level: process.env.LOG_LEVEL || "info",
    transport: {
        targets: [
            {
                target: "pino-pretty",
                options: { colorize: true }, // Pretty logs in the console
            },
            {
                target: "pino/file",
                options: { destination: "./logs.log" }, // Save logs to a file
            }
        ]
    }
});

export default logger;
```

Now, when you run:
```sh
bun run app.ts
```
Logs will be saved in **logs.log** while still appearing in the console.

---

## **Step 7: Challenge**
- Modify the logger to dynamically change log levels using an environment variable.
- Run the app with `LOG_LEVEL=debug bun run app.ts` and observe extra debug logs.

---

## **Expected Learning Outcomes**
By completing this exercise, students will:
✅ Set up logging using **Bun** and **Pino** effortlessly.  
✅ Understand log levels (`info`, `warn`, `error`, `debug`, etc.).  
✅ Learn how to write logs to a **file**.  
✅ Gain experience configuring logs dynamically using **environment variables**.  

---

# Week 11: Monitoring and Logging with Datadog

## Objective

Learn how to add **monitoring and logging** to a Bun application using **Datadog** — one of the most widely used observability platforms in industry. By the end, you'll have structured JSON logging with **Pino**, custom application metrics sent via **DogStatsD**, and a live dashboard in Datadog — all running locally with a Datadog Agent in Docker. You'll understand why monitoring matters, what to measure, and how a real-world monitoring platform works.

---

## Why Monitoring?

You've built an application, containerized it with Docker, orchestrated it with Kubernetes, and provisioned infrastructure with Terraform. But once your app is running in production, how do you know if it's actually working?

Without monitoring, you learn about problems from **angry users**. With monitoring, you know before they do.

Monitoring serves three purposes:

- **Business** — Are users able to use the product? Are key flows working?
- **Incident response** — Something is broken right now. Where? Why?
- **Historical analysis** — What changed? When did performance degrade? Can we predict failures?

---

## What is Monitoring?

Monitoring is the practice of collecting, processing, and displaying data about your application's behavior. It covers four key areas:

| Area | What It Means | Tools We'll Use |
|---|---|---|
| **Instrument** | Add code that emits data (logs, metrics) | Pino (logging), hot-shots (metrics) |
| **Store** | Collect and persist the data | Datadog Agent → Datadog platform |
| **Visualize** | Create dashboards to see what's happening | Datadog dashboards |
| **Alert** | Get notified when something goes wrong | Datadog monitors |

### Logging vs. Metrics

These are two complementary approaches to observability:

| | Logging | Metrics |
|---|---|---|
| What it captures | Individual events with context | Numeric measurements over time |
| Example | `"User 123 failed to login: invalid password"` | `login_failures_total: 47` |
| Storage cost | High (every event stored) | Low (aggregated numbers) |
| Best for | Debugging specific issues | Detecting trends and alerting |
| Query style | "What happened to user 123?" | "How many errors per minute?" |

You need **both**. Metrics tell you _something is wrong_. Logs tell you _what went wrong_.

### Logging Levels

Not all log messages are equal. Log levels control which messages are recorded:

| Level | When to Use | Example |
|---|---|---|
| `debug` | Detailed info for developers during development | `"Parsed request body: {name: 'Alice'}"` |
| `info` | Normal operations worth recording | `"Server started on port 3000"` |
| `warn` | Something unexpected but not broken | `"API response slow: 2300ms"` |
| `error` | Something failed but the app continues | `"Database query failed: connection timeout"` |
| `fatal` | The app cannot continue | `"Unable to bind to port 3000, shutting down"` |

**Important rule:** In production, the default log level is typically `warn` or `info`. Setting it to `debug` in production would flood your logs with noise and increase storage costs. You raise the level to see less, lower it to see more.

### Dashboard Health: Green / Amber / Red

Dashboards typically use a traffic-light system:

| Color | Meaning | Action |
|---|---|---|
| **Green** | Everything is operating normally | None — keep monitoring |
| **Amber** | Something is degraded or approaching a threshold | Investigate soon |
| **Red** | Something is broken or a critical threshold is breached | Act immediately |

### What is Datadog?

**Datadog** is a commercial monitoring and observability platform used by companies like Airbnb, Samsung, and Peloton. It combines metrics, logs, and traces in a single platform with a powerful query language and built-in alerting.

| Feature | What It Does |
|---|---|
| Infrastructure monitoring | Track CPU, memory, disk, network across all your hosts |
| Application metrics | Custom counters, gauges, histograms via DogStatsD |
| Log management | Centralized, searchable logs with automatic parsing |
| APM (Application Performance Monitoring) | Distributed tracing across services |
| Dashboards | Drag-and-drop dashboard builder with real-time data |
| Monitors & alerts | Threshold-based alerts with Slack, email, PagerDuty integration |

For this exercise, we'll use a **free 14-day trial** — no credit card required to sign up.

---

## Prerequisites

- **Docker** installed and running (Docker Desktop or Rancher Desktop)
- **Bun** installed (`curl -fsSL https://bun.sh/install | bash`)
- A **Datadog account** (you'll create one in Part 1)
- Familiarity with HTTP servers (from earlier weeks)

---

## Project Structure (What You'll Build)

```
week-11-monitoring/
├── src/
│   └── index.ts               ← Bun HTTP server with logging and metrics
├── .env                       ← Your Datadog API key (not committed to Git)
├── .gitignore                 ← Ignores .env and node_modules
├── docker-compose.yml         ← Datadog Agent service
├── package.json
└── tsconfig.json
```

You'll build a Bun HTTP server that logs every request with structured JSON and sends custom metrics to a local Datadog Agent via DogStatsD. The Agent forwards everything to the Datadog platform where you'll build dashboards and alerts.

---

## Part 1: Set Up Datadog

### Exercise 1: Create a Datadog Account and Get Your API Key

1. Go to [https://www.datadoghq.com](https://www.datadoghq.com) and click **Get Started Free**.

2. Sign up with your email. Select the **US1** region (or the region closest to you) when prompted.

3. Once logged in, get your API key:
   - Click your avatar (bottom-left) → **Organization Settings**
   - Go to **API Keys**
   - Copy your API key (it looks like a long hex string)

4. Create your project directory now (we'll set up the rest in Part 2):
   ```bash
   mkdir week-11-monitoring && cd week-11-monitoring
   ```

5. Create a `.env` file in the project root and add your API key:

   ```bash
   DD_API_KEY=your-api-key-here
   ```

   Replace `your-api-key-here` with the key you copied.

6. Create a `.gitignore` to make sure the key is never committed:

   ```
   node_modules
   .env
   ```

7. Verify the `.env` file:
   ```bash
   cat .env
   ```

   You should see your API key. **Never commit `.env` to Git** — the `.gitignore` entry protects you from doing this accidentally.

---

## Part 2: Set Up the Project

### Exercise 2: Create the Application

1. Make sure you're in the `week-11-monitoring` directory (created in Exercise 1) and initialize it:
   ```bash
   bun init -y
   ```

2. Create the directory structure:
   ```bash
   mkdir -p src
   ```

3. Create `src/index.ts` — a basic HTTP server with a few routes:

   ```typescript
   const server = Bun.serve({
     port: 3000,
     fetch(req) {
       const url = new URL(req.url);

       if (url.pathname === "/") {
         return new Response("Hello from monitored app!");
       }

       if (url.pathname === "/slow") {
         // Simulate a slow response (100-500ms)
         const delay = Math.floor(Math.random() * 400) + 100;
         return new Promise((resolve) =>
           setTimeout(
             () => resolve(new Response(`Responded in ${delay}ms`)),
             delay
           )
         );
       }

       if (url.pathname === "/error") {
         return new Response("Something went wrong", { status: 500 });
       }

       return new Response("Not Found", { status: 404 });
     },
   });

   console.log(`Server running at http://localhost:${server.port}`);
   ```

4. Run the server:
   ```bash
   bun run src/index.ts
   ```

5. Test it in another terminal:
   ```bash
   curl http://localhost:3000
   curl http://localhost:3000/slow
   curl http://localhost:3000/error
   curl http://localhost:3000/nonexistent
   ```

   You should see responses from each route. Press `Ctrl+C` to stop the server.

   Right now, the only output is `console.log` — you have no structured data, no log levels, no way to search or filter. Let's fix that.

---

## Part 3: Structured Logging with Pino

### Exercise 3: Add Pino for Structured Logging

**Pino** is a fast, structured JSON logger for Node.js and Bun. Instead of plain text, it outputs JSON objects that are easy to parse, search, and filter — exactly what Datadog's log management expects.

1. Install Pino and pino-pretty (a dev tool that makes JSON logs human-readable):
   ```bash
   bun add pino pino-pretty
   ```

2. Update `src/index.ts` to use Pino:

   ```typescript
   import pino from "pino";

   const logger = pino({
     level: process.env.LOG_LEVEL || "info",
     ...(process.env.NODE_ENV !== "production" && {
       transport: {
         target: "pino-pretty",
         options: { colorize: true },
       },
     }),
   });

   const server = Bun.serve({
     port: 3000,
     fetch(req) {
       const url = new URL(req.url);
       const start = Date.now();

       let response: Response;

       if (url.pathname === "/") {
         response = new Response("Hello from monitored app!");
       } else if (url.pathname === "/slow") {
         const delay = Math.floor(Math.random() * 400) + 100;
         const result = new Promise<Response>((resolve) =>
           setTimeout(() => {
             const res = new Response(`Responded in ${delay}ms`);
             const duration = Date.now() - start;
             logger.info(
               { method: req.method, path: url.pathname, status: 200, duration },
               "request completed"
             );
             resolve(res);
           }, delay)
         );
         return result;
       } else if (url.pathname === "/error") {
         logger.error(
           { method: req.method, path: url.pathname, status: 500 },
           "internal server error"
         );
         return new Response("Something went wrong", { status: 500 });
       } else {
         logger.warn(
           { method: req.method, path: url.pathname, status: 404 },
           "route not found"
         );
         return new Response("Not Found", { status: 404 });
       }

       const duration = Date.now() - start;
       logger.info(
         { method: req.method, path: url.pathname, status: 200, duration },
         "request completed"
       );
       return response;
     },
   });

   logger.info({ port: server.port }, "server started");
   ```

   **What changed:**

   | Before | After |
   |---|---|
   | `console.log("Server running...")` | `logger.info({ port: 3000 }, "server started")` |
   | No request logging | Every request logged with method, path, status, duration |
   | Plain text | Structured JSON (with pretty-printing in development) |

3. Run the server:
   ```bash
   bun run src/index.ts
   ```

   You should see a pretty-printed startup message.

4. Make some requests and watch the logs:
   ```bash
   curl http://localhost:3000
   curl http://localhost:3000/slow
   curl http://localhost:3000/error
   curl http://localhost:3000/nonexistent
   ```

   Each request produces a structured log line with the method, path, status code, and duration. Notice:
   - The `/` and `/slow` routes log at `info` level
   - The `/error` route logs at `error` level
   - The `/nonexistent` route logs at `warn` level

5. Now see what production output looks like. Stop the server and run it with:
   ```bash
   NODE_ENV=production bun run src/index.ts
   ```

   Make a request — you'll see raw JSON output (no colors, no formatting). This JSON format is exactly what Datadog expects when ingesting logs. Stop the server.

### Exercise 4: Control Log Levels

1. Update the logger creation in `src/index.ts` to read a `LOG_LEVEL` environment variable (it already does from Exercise 3):

   ```typescript
   const logger = pino({
     level: process.env.LOG_LEVEL || "info",
     // ... rest of config
   });
   ```

2. Add a debug log line. Add this inside the `"/"` route handler, before creating the response:

   ```typescript
   if (url.pathname === "/") {
     logger.debug({ headers: Object.fromEntries(req.headers) }, "incoming request headers");
     response = new Response("Hello from monitored app!");
   }
   ```

3. Run with the default level (`info`):
   ```bash
   bun run src/index.ts
   ```

   Hit `curl http://localhost:3000` — you'll see the `info` log but **not** the `debug` log.

4. Run with `debug` level:
   ```bash
   LOG_LEVEL=debug bun run src/index.ts
   ```

   Hit the same endpoint — now you see the debug log with all the request headers. This is very useful during development or when troubleshooting a production issue (temporarily lower the level, then raise it back).

   **The level hierarchy:** `debug` < `info` < `warn` < `error` < `fatal`. Setting the level to `warn` means only `warn`, `error`, and `fatal` messages are logged.

---

## Part 4: Run the Datadog Agent

### Exercise 5: Start the Datadog Agent with Docker Compose

The **Datadog Agent** is a process that runs alongside your application. It collects metrics and logs and forwards them to the Datadog platform. We'll run it as a Docker container.

1. Create `docker-compose.yml`:

   ```yaml
   services:
     datadog-agent:
       image: gcr.io/datadoghq/agent:7
       env_file:
         - .env
       environment:
         - DD_SITE=datadoghq.com
         - DD_DOGSTATSD_NON_LOCAL_TRAFFIC=true
         - DD_APM_ENABLED=false
       ports:
         - "8125:8125/udp"
       volumes:
         - /var/run/docker.sock:/var/run/docker.sock:ro
   ```

   **What each setting does:**

   | Setting | Purpose |
   |---|---|
   | `env_file: .env` | Loads `DD_API_KEY` from your `.env` file to authenticate with Datadog |
   | `DD_SITE` | Which Datadog region to send data to (change if you chose a different region) |
   | `DD_DOGSTATSD_NON_LOCAL_TRAFFIC` | Accept metrics from outside the container (from your Bun app on the host) |
   | `DD_APM_ENABLED=false` | Disable APM tracing (we'll focus on metrics and logs) |
   | Port `8125/udp` | DogStatsD port — where your app sends custom metrics |
   | Docker socket volume | Lets the Agent collect Docker container metrics |

   > **Note:** If you're on macOS and the Docker socket is at a different path (e.g., Rancher Desktop uses `~/.rd/docker.sock`), update the volume accordingly, or remove the `volumes` section entirely — it's only needed for Docker container metrics.

2. Start the Agent:
   ```bash
   docker compose up -d
   ```

3. Verify the Agent is running:
   ```bash
   docker compose logs datadog-agent | tail -20
   ```

   Look for a line like `"Successfully posted payload to Datadog"` or similar. It may take 15-30 seconds for the first successful post.

4. Check in the Datadog UI:
   - Go to [https://app.datadoghq.com](https://app.datadoghq.com)
   - Navigate to **Infrastructure → Host Map**
   - You should see your machine appear as a host within a few minutes

   If you don't see it, check that your `.env` file has the correct API key:
   ```bash
   cat .env
   docker compose logs datadog-agent | grep -i "error\|invalid\|api"
   ```

---

## Part 5: Custom Metrics with DogStatsD

### Exercise 6: Send Metrics from Your App

**DogStatsD** is a metrics aggregation service built into the Datadog Agent. Your app sends metrics over UDP to port 8125, and the Agent aggregates and forwards them to Datadog. The `hot-shots` npm package is a Node.js/Bun client for DogStatsD.

1. Install hot-shots:
   ```bash
   bun add hot-shots
   ```

2. Replace `src/index.ts` with a version that sends metrics to DogStatsD:

   ```typescript
   import pino from "pino";
   import StatsD from "hot-shots";

   // --- Logger ---
   const logger = pino({
     level: process.env.LOG_LEVEL || "info",
     ...(process.env.NODE_ENV !== "production" && {
       transport: {
         target: "pino-pretty",
         options: { colorize: true },
       },
     }),
   });

   // --- Metrics ---
   const metrics = new StatsD({
     host: process.env.DD_AGENT_HOST || "localhost",
     port: 8125,
     prefix: "bun_app.",
     errorHandler(error) {
       logger.error({ error: error.message }, "StatsD error");
     },
   });

   // --- Server ---
   const server = Bun.serve({
     port: 3000,
     async fetch(req) {
       const url = new URL(req.url);
       const start = Date.now();
       const requestId = crypto.randomUUID();
       const reqLogger = logger.child({ requestId });

       let response: Response;
       let status: number;

       if (url.pathname === "/") {
         reqLogger.debug(
           { headers: Object.fromEntries(req.headers) },
           "incoming request headers"
         );
         response = new Response("Hello from monitored app!");
         status = 200;
       } else if (url.pathname === "/slow") {
         const delay = Math.floor(Math.random() * 400) + 100;
         await new Promise((resolve) => setTimeout(resolve, delay));
         response = new Response(`Responded in ${delay}ms`);
         status = 200;
       } else if (url.pathname === "/error") {
         response = new Response("Something went wrong", { status: 500 });
         status = 500;
       } else {
         response = new Response("Not Found", { status: 404 });
         status = 404;
       }

       // Record metrics
       const duration = Date.now() - start;
       const tags = [
         `method:${req.method}`,
         `path:${url.pathname}`,
         `status:${status}`,
       ];

       metrics.increment("http.requests.total", 1, tags);
       metrics.histogram("http.request.duration", duration, tags);

       // Log the request
       const logData = {
         method: req.method,
         path: url.pathname,
         status,
         duration: `${duration}ms`,
       };

       if (status >= 500) {
         reqLogger.error(logData, "request failed");
       } else if (status >= 400) {
         reqLogger.warn(logData, "client error");
       } else {
         reqLogger.info(logData, "request completed");
       }

       return response;
     },
   });

   logger.info({ port: server.port }, "server started");
   ```

   **What's new:**

   | Concept | What It Does |
   |---|---|
   | `StatsD` client | Connects to the Datadog Agent's DogStatsD service on UDP port 8125 |
   | `prefix: "bun_app."` | All metric names will start with `bun_app.` (e.g., `bun_app.http.requests.total`) |
   | `metrics.increment()` | Sends a **counter** metric — total requests, broken down by tags |
   | `metrics.histogram()` | Sends a **histogram** metric — request duration with automatic percentiles |
   | `tags` | Dimensions to slice metrics by — method, path, status code |
   | `requestId` | UUID attached to every log line for tracing specific requests |

3. Make sure the Datadog Agent is running (from Exercise 5), then start the server:
   ```bash
   bun run src/index.ts
   ```

4. Generate some traffic:
   ```bash
   for i in $(seq 1 20); do curl -s http://localhost:3000 > /dev/null; done
   for i in $(seq 1 10); do curl -s http://localhost:3000/slow > /dev/null; done
   for i in $(seq 1 5); do curl -s http://localhost:3000/error > /dev/null; done
   ```

5. Check that the Agent is receiving metrics:
   ```bash
   docker compose exec datadog-agent agent status | grep -A 5 "DogStatsD"
   ```

   You should see a section showing DogStatsD is running and has processed packets.

---

## Part 6: Explore Metrics in Datadog

### Exercise 7: Find Your Metrics in the Datadog UI

Metrics take 1-2 minutes to appear in the Datadog UI after the Agent sends them.

1. Go to [https://app.datadoghq.com](https://app.datadoghq.com).

2. Navigate to **Metrics → Explorer**.

3. In the metric search box, type `bun_app.http.requests.total` and select it.

4. You should see your request counts. Click **Add Query** to explore further:

   **Total requests by path:**
   - Metric: `bun_app.http.requests.total`
   - Group by: `path`

   **Error rate (5xx responses only):**
   - Metric: `bun_app.http.requests.total`
   - Filter: `status:500`

   **95th percentile request duration:**
   - Metric: `bun_app.http.request.duration.95percentile`
   - Group by: `path`

   DogStatsD histograms automatically generate multiple metrics:

   | Generated Metric | What It Measures |
   |---|---|
   | `bun_app.http.request.duration.avg` | Average request duration |
   | `bun_app.http.request.duration.max` | Maximum request duration |
   | `bun_app.http.request.duration.median` | Median (p50) request duration |
   | `bun_app.http.request.duration.95percentile` | 95th percentile |
   | `bun_app.http.request.duration.count` | Number of data points |

5. Generate more traffic to see the graphs update:
   ```bash
   while true; do curl -s http://localhost:3000 > /dev/null; curl -s http://localhost:3000/slow > /dev/null; curl -s http://localhost:3000/error > /dev/null; sleep 0.5; done
   ```

   Press `Ctrl+C` to stop after 30 seconds or so.

### Exercise 8: Build a Dashboard

1. In Datadog, go to **Dashboards → New Dashboard**.

2. Give it a name: "Bun App Monitoring".

3. **Widget 1 — Request Rate:**
   - Click **Add Widgets** → drag **Timeseries** onto the dashboard
   - Metric: `bun_app.http.requests.total`
   - Display as: `as rate()`  (click the metric and select "as Rate")
   - Group by: `path`
   - Title: "Request Rate (req/s)"
   - Click **Save**

4. **Widget 2 — Error Rate:**
   - Add another **Timeseries** widget
   - Metric: `bun_app.http.requests.total`
   - Filter: `status:500`
   - Display as: `as rate()`
   - Title: "Error Rate (5xx/s)"
   - Click **Save**

5. **Widget 3 — Response Time (p95):**
   - Add another **Timeseries** widget
   - Metric: `bun_app.http.request.duration.95percentile`
   - Group by: `path`
   - Title: "Response Time p95 (ms)"
   - Click **Save**

6. **Widget 4 — Total Requests:**
   - Add a **Query Value** widget
   - Metric: `bun_app.http.requests.total` with `as count()` aggregation
   - Title: "Total Requests"
   - Click **Save**

7. Arrange the widgets by dragging them. Your dashboard now shows the same information a production team would use to monitor a live service.

8. Generate traffic to see the dashboard come alive:
   ```bash
   while true; do curl -s http://localhost:3000 > /dev/null; curl -s http://localhost:3000/slow > /dev/null; curl -s http://localhost:3000/error > /dev/null; sleep 0.5; done
   ```

   Watch the Datadog dashboard update. Press `Ctrl+C` to stop after a minute.

---

## Part 7: Monitors and Alerts

### Exercise 9: Create a Datadog Monitor

Dashboards are useful, but you can't watch them 24/7. **Monitors** notify you when something crosses a threshold — this is the "Red" in the Green/Amber/Red system.

1. In Datadog, go to **Monitors → New Monitor**.

2. Select **Metric** as the monitor type.

3. **Define the metric:**
   - Metric: `bun_app.http.requests.total`
   - Filter: `status:500`
   - Set the evaluation to: **sum** of the metric **as rate** over the last **1 minute**

4. **Set alert conditions:**
   - **Alert threshold**: 0.5 (fire when error rate exceeds 0.5 per second)
   - **Warning threshold**: 0.1 (warn when approaching the limit)

   This maps to the Green/Amber/Red system:
   - Below 0.1 → **Green** (normal)
   - Between 0.1 and 0.5 → **Amber** (warning)
   - Above 0.5 → **Red** (alert)

5. **Name the monitor:** "High Error Rate — Bun App"

6. **Configure notifications:**
   - For this exercise, you can skip notification channels (Slack, email, PagerDuty)
   - In a real setup, this is where you'd route alerts to the on-call team

7. Click **Create**.

8. Generate errors to trigger the monitor:
   ```bash
   for i in $(seq 1 100); do curl -s http://localhost:3000/error > /dev/null; done
   ```

9. Go to **Monitors → Manage Monitors**. Within a few minutes, you should see your monitor transition from **OK** → **Warn** → **Alert** as the error rate crosses the thresholds.

   In production, this alert would page someone, send a Slack message, or create an incident. The value of monitoring is that problems are detected _automatically_ — no one needs to be watching a dashboard.

---

## Part 8: Application Metrics — Gauges

### Exercise 10: Track Active Connections

So far we've used **counters** (always increase) and **histograms** (track distributions). A **gauge** is a metric that can go up _and_ down — perfect for tracking things like active connections, queue depth, or cache size.

1. Add a gauge to `src/index.ts`. Add this after the `StatsD` client creation:

   ```typescript
   // Track active connections with a gauge
   let activeConnections = 0;
   ```

2. Update the `fetch` handler to track active connections. Add at the start of the function (after `const reqLogger`):

   ```typescript
   activeConnections++;
   metrics.gauge("http.active_connections", activeConnections);
   ```

3. Add at the end of the function, just before `return response;`:

   ```typescript
   activeConnections--;
   metrics.gauge("http.active_connections", activeConnections);
   ```

4. Restart the server and generate concurrent requests to see the gauge in action:
   ```bash
   bun run src/index.ts
   ```

   In another terminal:
   ```bash
   for i in $(seq 1 20); do curl -s http://localhost:3000/slow & done
   wait
   ```

   The `/slow` route takes 100-500ms, so multiple requests will be in-flight at the same time. The `active_connections` gauge will rise and fall.

5. In Datadog, go to **Metrics → Explorer** and search for `bun_app.http.active_connections`. You should see the gauge spike during concurrent requests and drop back to 0 when they complete.

   **Metric type summary:**

   | Type | What It Does | Example | DogStatsD Method |
   |---|---|---|---|
   | Counter | Only increases | Total requests | `metrics.increment()` |
   | Gauge | Can increase or decrease | Active connections | `metrics.gauge()` |
   | Histogram | Tracks distributions | Request duration | `metrics.histogram()` |

---

## Challenge Exercises

### Challenge 1: Service Check

1. Add a `/health` endpoint to your app:
   ```typescript
   if (url.pathname === "/health") {
     return Response.json({
       status: "ok",
       uptime: process.uptime(),
       timestamp: new Date().toISOString(),
     });
   }
   ```
2. Use the DogStatsD `serviceCheck` method to report the app's health:
   ```typescript
   metrics.serviceCheck("bun_app.health", StatsD.CHECKS.OK, {
     message: "App is healthy",
   });
   ```
3. Send a `CRITICAL` service check when the error rate is too high
4. View the service check in Datadog under **Monitors → Check Summary**

### Challenge 2: Custom Events

1. Use `metrics.event()` to send a Datadog event when your app starts:
   ```typescript
   metrics.event("App Started", `Bun app started on port ${server.port}`, {
     alert_type: "info",
   });
   ```
2. Send a `warning` event when a request takes longer than 1 second
3. Send an `error` event when 10 consecutive errors occur
4. View events in Datadog under **Events → Explorer**

### Challenge 3: Tagged Dashboards

1. Add a `version` tag to all your metrics:
   ```typescript
   const tags = [
     `method:${req.method}`,
     `path:${url.pathname}`,
     `status:${status}`,
     `version:1.0.0`,
   ];
   ```
2. Change the version to `2.0.0` and restart the app
3. In Datadog, create a dashboard that compares metrics between versions using the `version` tag
4. This is how teams track the impact of new deployments in production

---

## Quick Reference

### Pino Logger

| Method | Level | When to Use |
|---|---|---|
| `logger.debug(data, msg)` | 20 | Detailed debugging information |
| `logger.info(data, msg)` | 30 | Normal operational events |
| `logger.warn(data, msg)` | 40 | Unexpected but non-critical issues |
| `logger.error(data, msg)` | 50 | Errors that need attention |
| `logger.fatal(data, msg)` | 60 | Unrecoverable errors |
| `logger.child({ key: val })` | — | Create a child logger with extra context |

### DogStatsD Metric Types (hot-shots)

| Method | Metric Type | What It Does | Example |
|---|---|---|---|
| `metrics.increment(name, value, tags)` | Counter | Count occurrences | Total requests, errors |
| `metrics.gauge(name, value, tags)` | Gauge | Current value (up/down) | Active connections, queue size |
| `metrics.histogram(name, value, tags)` | Histogram | Value distribution | Request duration, payload size |
| `metrics.set(name, value, tags)` | Set | Count unique values | Unique users, unique IPs |
| `metrics.event(title, text, options)` | Event | Notable occurrence | Deployments, errors |
| `metrics.serviceCheck(name, status)` | Service Check | Health status | App health, DB connectivity |

### Datadog Agent Environment Variables

| Variable | Purpose | Example |
|---|---|---|
| `DD_API_KEY` | Authenticate with Datadog | `abc123...` |
| `DD_SITE` | Datadog region | `datadoghq.com`, `datadoghq.eu` |
| `DD_DOGSTATSD_NON_LOCAL_TRAFFIC` | Accept external DogStatsD traffic | `true` |
| `DD_APM_ENABLED` | Enable/disable APM tracing | `true` / `false` |
| `DD_LOG_LEVEL` | Agent log verbosity | `info`, `debug`, `warn` |

### Docker Compose Commands

| Command | Description |
|---|---|
| `docker compose up -d` | Start services in background |
| `docker compose down` | Stop and remove services |
| `docker compose logs -f` | Follow logs from all services |
| `docker compose exec datadog-agent agent status` | Check Agent status |
| `docker compose ps` | List running services |

### Datadog UI Navigation

| Section | URL Path | What You'll Find |
|---|---|---|
| Metrics Explorer | `/metric/explorer` | Query and graph any metric |
| Dashboards | `/dashboard` | Custom dashboard builder |
| Monitors | `/monitors` | Alert rules and status |
| Infrastructure | `/infrastructure` | Host map, processes, containers |
| Events | `/event/explorer` | Event timeline |

---

## Next Steps

After completing these exercises, explore:
- **Datadog APM** — Distributed tracing to follow requests across microservices (requires `dd-trace` with Node.js, or OpenTelemetry with Bun)
- **Datadog Log Management** — Send structured logs to Datadog for centralized search and analysis
- **Sentry** — Error tracking focused on catching and diagnosing exceptions with stack traces
- **OpenTelemetry** — A vendor-neutral standard for collecting traces, metrics, and logs that works with Datadog, Grafana, and other platforms
- **Synthetic Monitoring** — Datadog can run automated tests against your endpoints from locations worldwide

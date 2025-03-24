# 🧪 In-Class Exercise: Application Monitoring with Bun and TypeScript

## 🎯 Objective
Learn how to **track key metrics** in a Bun + TypeScript server _without using logs_. You’ll:
- Track request counts
- Measure response times
- Expose metrics via a `/metrics` endpoint
- (Bonus) Format output for Prometheus

---

## 📦 Step 1: Initialize the Project

```sh
bun init
```
Choose **TypeScript** when prompted.

---

## 🛠️ Step 2: Build the HTTP Server with Metrics

Create a file called `server.ts`:

```ts
const metrics = {
    requestCount: 0,
    totalResponseTime: 0,
  };
  
  const server = Bun.serve({
    port: 3000,
    async fetch(req: Request): Promise<Response> {
      const start = Date.now();
      metrics.requestCount++;
  
      if (req.url.endsWith("/metrics")) {
        const avgResponseTime =
          metrics.requestCount === 0
            ? 0
            : metrics.totalResponseTime / metrics.requestCount;
  
        const body = `
  # HELP http_requests_total Total number of HTTP requests
  # TYPE http_requests_total counter
  http_requests_total ${metrics.requestCount}
  
  # HELP http_response_time_average_ms Average response time in milliseconds
  # TYPE http_response_time_average_ms gauge
  http_response_time_average_ms ${avgResponseTime.toFixed(2)}
        `.trim();
  
        return new Response(body, {
          headers: { "Content-Type": "text/plain" },
        });
      }
  
      // Simulate work
      await new Promise((res) => setTimeout(res, Math.random() * 300));
  
      const duration = Date.now() - start;
      metrics.totalResponseTime += duration;
  
      return new Response("Hello, world!");
    },
  });
  
  console.log(`✅ Server running at http://localhost:${server.port}`);
```

---

## 🚀 Step 3: Run It

```sh
bun run server.ts
```

Now try these in your browser or `curl`:
- `http://localhost:3000` → ✅ Returns: "Hello, world!"
- `http://localhost:3000/metrics` → 📊 Returns: Prometheus-style metrics

---

## 📈 Sample `/metrics` Output

```txt
# HELP http_requests_total Total number of HTTP requests
# TYPE http_requests_total counter
http_requests_total 5

# HELP http_response_time_average_ms Average response time in milliseconds
# TYPE http_response_time_average_ms gauge
http_response_time_average_ms 112.25
```

This format is compatible with Prometheus scrapers.

---

## 🧠 Optional Stretch Challenge

Enhance your metrics:
- `successfulRequests`
- `failedRequests`
- `lastRequestTimestamp`
- `uptimeSeconds`

Example:
```ts
const serverStartTime = Date.now();
uptimeSeconds: Math.floor((Date.now() - serverStartTime) / 1000)
```

---

## ✅ Learning Outcomes

| ✅ You Will Learn To:                |
|-------------------------------------|
| Understand the difference between logs & metrics |
| Track basic server statistics manually |
| Expose metrics in a Prometheus-readable format |
| Use Bun’s built-in HTTP server with TypeScript |

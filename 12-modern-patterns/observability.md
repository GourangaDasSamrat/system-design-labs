# 🔭 Observability — Logs, Metrics, Tracing

> **One-liner:** Observability is your ability to understand what's happening inside your system from the outside — the three pillars are logs, metrics, and distributed traces.

> 🎨 **Diagram:** [observability.excalidraw](../diagrams/observability.excalidraw) — open in [Excalidraw](https://excalidraw.com) (File → Open)

---

## 🏛️ The Three Pillars

```
┌──────────────────────────────────────────────────────────────┐
│                       OBSERVABILITY                          │
│                                                              │
│  📋 LOGS          📊 METRICS           🔗 TRACES             │
│  "What happened"  "How is it doing"    "Where did time go"   │
│  Events, errors   Numbers over time    Request journey        │
└──────────────────────────────────────────────────────────────┘
```

---

## 📋 Pillar 1: Logs

Timestamped records of discrete events.

### Structured Logging (JSON — do this, not plain text)

```json
{
  "timestamp": "2025-03-15T10:23:45Z",
  "level": "ERROR",
  "service": "payment-service",
  "traceId": "abc-123-xyz",
  "userId": 42,
  "message": "Payment declined",
  "code": "INSUFFICIENT_FUNDS",
  "amount": 99.99,
  "durationMs": 342
}
```

**Why structured?** You can filter, aggregate, and alert on specific fields. `grep` on plain text doesn't scale to millions of log lines.

### Log Levels

| Level   | When to Use                              |
| ------- | ---------------------------------------- |
| `DEBUG` | Dev only — verbose, turned off in prod   |
| `INFO`  | Normal operations, key business events   |
| `WARN`  | Unexpected but handled (retry succeeded) |
| `ERROR` | Something failed, needs attention        |
| `FATAL` | Service cannot continue, immediate alert |

### Log Aggregation Stack

```
Services → Fluentd/Logstash → Elasticsearch → Kibana (ELK)
                           or
Services → CloudWatch Logs (AWS) → CloudWatch Insights
Services → Loki → Grafana (cheaper, label-based)
```

---

## 📊 Pillar 2: Metrics

Numerical measurements aggregated over time.

### The Four Golden Signals (Google SRE)

| Signal         | What It Measures       | Example Alert   |
| -------------- | ---------------------- | --------------- |
| **Latency**    | How long requests take | p99 > 500ms     |
| **Traffic**    | How much demand        | RPS spike >2x   |
| **Errors**     | Rate of failures       | Error rate > 1% |
| **Saturation** | How full is the system | CPU > 80%       |

### Key Metrics to Track

```
# Business
orders_per_minute
revenue_per_hour
active_users

# Infrastructure
http_request_duration_seconds (histogram)
http_requests_total{status="500"} (counter)
db_connection_pool_used (gauge)
cache_hit_rate (gauge)
message_queue_depth (gauge)
```

### Metrics Stack

```
Services → Prometheus (pull-based scraping) → Grafana (dashboards)
                                           → AlertManager (alerts)

OR: Datadog, New Relic, CloudWatch Metrics (SaaS, easier ops)
```

---

## 🔗 Pillar 3: Distributed Tracing

Follow a single request across multiple services.

### The Problem Without Tracing

```
User: "The checkout is slow"
You:  Is it the API gateway? Auth service? Cart service?
      Payment service? DB? Which hop is slow? No idea. 😰
```

### With Tracing

```
Trace ID: abc-123

API Gateway      ████ 5ms
  └─ Auth        ████ 12ms
  └─ Cart        ████████████ 250ms  ← bottleneck!
       └─ DB     █████████ 220ms    ← query too slow
  └─ Payment     ████ 8ms

Total: 275ms
```

### How It Works

```javascript
// Middleware auto-injects trace context
app.use((req, res, next) => {
  const traceId = req.headers["x-trace-id"] || generateId();
  const spanId = generateId();

  // Pass downstream in headers:
  // x-trace-id: abc-123
  // x-parent-span-id: span-456
  req.traceContext = { traceId, spanId };
  next();
});
```

### Tracing Tools

| Tool              | Type        | Best For                       |
| ----------------- | ----------- | ------------------------------ |
| **Jaeger**        | Open source | Kubernetes, self-hosted        |
| **Zipkin**        | Open source | Simpler setup                  |
| **AWS X-Ray**     | Managed     | AWS native workloads           |
| **Datadog APM**   | SaaS        | Full observability suite       |
| **OpenTelemetry** | Standard    | Vendor-neutral instrumentation |

**OpenTelemetry** is the industry standard for instrumentation — use it and swap backends freely.

---

## 🔔 Alerting Strategy

```
Tier 1 (Page immediately):  p99 latency > 2s for 5 min
                            Error rate > 5%
                            Service down

Tier 2 (Slack/ticket):      Error rate > 1%
                            p99 > 500ms
                            DB connections > 80%

Tier 3 (Weekly review):     Slow trend upward
                            Disk usage > 70%
```

**Alert on symptoms, not causes.** Alert on "error rate high", not "CPU high" — users don't care about CPU.

---

## ✅ Pros

- Find bugs and incidents fast (MTTR reduction)
- Capacity planning from metrics trends
- Proves SLA compliance with data
- Tracing reveals hidden performance bottlenecks

## ❌ Cons

- Adds cost: storage for logs and metrics, SaaS fees
- Cardinality explosion in metrics (too many label combinations)
- Log volume can be enormous at scale (sampling helps)
- Requires cultural buy-in — teams must actually look at dashboards

## ⚖️ When to Use / When NOT to Use

**✅ Use when:**

- Any production system (non-negotiable at scale)
- Microservices where request paths cross multiple services
- SLA commitments requiring uptime/latency guarantees
- Debugging production incidents

**❌ Skip the complexity when:**

- Early prototype / dev environment
- Single-service monolith with few users (basic logging is fine)
- Cost is the primary constraint (use sampling + retention policies)

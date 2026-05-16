# 💥 Failure Handling & Resilience

> Interviewers LOVE failure thinking 😏 — "What happens when X fails?" is the classic follow-up.

---

## The Mindset

> **Design for failure, not for success.**
> Every component WILL fail. The question is: does one failure cause total system failure, or does the system degrade gracefully?

---

## 🔁 Retry Strategies

### Naive Retry (❌ Don't do this)

```
Request fails → immediately retry → retry → retry → retry
```

- Hammers a struggling service → makes it worse
- Can cause **retry storms** (10k clients all retrying at once)

### Exponential Backoff (✅ Standard)

```
Attempt 1: fail → wait 1s
Attempt 2: fail → wait 2s
Attempt 3: fail → wait 4s
Attempt 4: fail → wait 8s → give up
```

- Wait time doubles each attempt
- Reduces load on recovering service
- Formula: `wait = baseDelay * 2^attempt`

### Exponential Backoff + Jitter (✅ Best)

```
wait = random(0, baseDelay * 2^attempt)
```

- Randomness prevents thundering herd (all clients retrying at exact same moment)
- AWS SDK uses this by default

### Retry Budget

- Cap total retries across all clients (e.g., max 10% of requests can be retries)
- Prevents cascading retry storms at scale

**What to retry:**

- ✅ Transient errors: network timeout, 429 (rate limited), 503 (service unavailable)
- ❌ Don't retry: 400 (bad request), 401 (auth), 404 (not found) — retrying won't help

---

## 📬 Dead Letter Queues (DLQ)

When a message in a queue fails processing repeatedly → route it to a DLQ instead of losing it.

```
SQS Queue → Consumer processes message
              → Success: delete from queue ✅
              → Fail (3 times): move to DLQ 📦
```

**DLQ allows:**

- Inspect failed messages to find bugs
- Replay messages after fixing the bug
- Alert on DLQ depth (metric = something is broken)

**Set up:**

- `maxReceiveCount` = how many times to attempt before DLQ
- Separate queue for dead letters (same type as source)
- CloudWatch alarm on `ApproximateNumberOfMessagesVisible` in DLQ

---

## 🌊 Graceful Degradation

When a non-critical dependency fails, the system keeps working in reduced capacity.

### Examples:

| Scenario                    | Bad (total failure) | Good (graceful)                              |
| --------------------------- | ------------------- | -------------------------------------------- |
| Recommendation service down | Whole page errors   | Show popular items instead                   |
| Reviews service down        | Product page errors | Hide review section, show product            |
| Payment provider slow       | Checkout hangs      | Show "processing, email confirmation coming" |
| Search index down           | 500 error           | Fall back to basic DB query                  |

**Key question in interview:** "Which parts of your system are critical vs. nice-to-have?"

- Critical: checkout, auth, core data reads
- Non-critical: recommendations, analytics, social features

### Feature Flags for Degradation

```
if (feature_enabled('recommendations')) {
  return await getRecommendations(userId);
} else {
  return getPopularItems(); // fallback
}
```

Toggle features off instantly without a deploy.

---

## 🐒 Chaos Engineering

Intentionally inject failures in production (or staging) to find weaknesses before real incidents do.

**Principle:** You don't know if your failover works until you've tested it.

### Process:

1. **Hypothesis:** "If the recommendations service goes down, users can still checkout"
2. **Inject:** Kill the recommendations service
3. **Observe:** What breaks? What degrades? What alerts fire?
4. **Fix:** Improve resilience based on findings
5. **Repeat**

### Tools:

- **Netflix Chaos Monkey** — randomly kills EC2 instances
- **Gremlin** — controlled chaos (network latency, CPU spike, pod kills)
- **AWS Fault Injection Simulator** — managed chaos on AWS

### Starting points (low risk):

- Kill a non-critical service and verify degradation works
- Add 200ms latency to a downstream call and verify timeouts fire
- Simulate a DB read replica failure — does traffic shift to primary?

---

## 🛡️ Bulkhead Pattern

Isolate components so one failure doesn't drain all shared resources.

```
Without bulkhead:
  Slow service B fills all 100 thread pool slots
  → Service A and C requests also fail (no threads left)

With bulkhead:
  Service A: 33 threads
  Service B: 33 threads  ← B fills its own threads, doesn't affect others
  Service C: 33 threads
```

Applied via: separate thread pools, separate connection pools, Kubernetes resource limits.

---

## ⏱️ Timeout Everything

Every network call must have a timeout. No exception.

- **Connection timeout:** how long to wait to establish connection (~5s)
- **Read timeout:** how long to wait for response after connecting (~30s)
- **Overall request timeout:** end-to-end budget per request

Without timeouts: one slow service causes thread starvation across the whole system.

---

## ⚖️ Trade-offs

✅ **Pros of resilience patterns**

- System stays up even when parts fail
- Better user experience under degraded conditions
- Faster incident recovery

❌ **Cons**

- More complex code (retry logic, fallbacks)
- More infrastructure (DLQs, circuit breakers)
- Chaos engineering requires cultural buy-in

⚖️ **Interview tip**
When you describe any component → follow up with: "And if this fails, here's what happens..." — this instantly signals senior thinking.

---

## 🎨 Diagram

See: [`diagrams/failure-resilience.excalidraw`](../diagrams/failure-resilience.excalidraw)

# 🚦 Rate Limiter Designs

> Rate limiting is a classic system design topic. Know ALL the algorithms — not just one.

---

## Why Rate Limit?

- Protect APIs from abuse (DDoS, scraping)
- Ensure fair usage across clients
- Protect downstream services from overload
- Enforce business rules (free tier: 100 req/day)

---

## Algorithm 1: Fixed Window Counter

```
Window: 00:00 → 01:00
Limit: 100 requests per hour

User sends request at 00:59:59 → counter = 100 → allowed ✅
User sends request at 01:00:01 → new window, counter = 1 → allowed ✅
```

**Problem: Boundary burst attack**

```
00:59 → 100 requests (at limit)
01:00 → 100 requests (new window reset)
Total: 200 requests in 2 seconds 💥
```

✅ Simple, low memory
❌ Allows 2x traffic at window boundaries

---

## Algorithm 2: Sliding Window Log

- Store timestamp of every request in a log (sorted set)
- On new request: remove timestamps older than window, count remaining
- If count < limit → allow

```
Limit: 5 requests per minute
Log: [00:00:10, 00:00:30, 00:00:45, 00:00:55]
New request at 00:01:20:
  Remove entries < 00:00:20 → log: [00:00:30, 00:00:45, 00:00:55]
  Count = 3, limit = 5 → allow ✅
  Add 00:01:20 to log
```

✅ Perfectly accurate, no boundary problem
❌ High memory — store every request timestamp
❌ Doesn't scale to millions of users

---

## Algorithm 3: Sliding Window Counter (Hybrid)

- Combine fixed window simplicity with sliding accuracy
- Use weighted count from current + previous window

```
Limit: 100 req/min
Previous window count: 80
Current window count: 30
Time into current window: 25% elapsed

Estimated rate = 80 * (1 - 0.25) + 30 = 90 → under limit ✅
```

✅ Memory efficient (only 2 counters)
✅ Good approximation (Cloudflare uses this)
❌ Slightly approximate, not exact

---

## Algorithm 4: Token Bucket 🪣

```
Bucket capacity: 10 tokens
Refill rate: 1 token/second

Each request consumes 1 token
If bucket empty → reject request

Bucket refills continuously at fixed rate
```

- Allows **bursts** up to bucket capacity
- Smooth average rate enforced over time
- ✅ Natural burst handling (users expect some burst tolerance)
- ✅ Low memory (capacity + current tokens + last refill time)
- ✅ Used by: AWS API Gateway, Stripe, most cloud providers

**Implementation with Redis:**

```python
tokens = redis.get(key)
now = time.time()
tokens = min(capacity, tokens + (now - last_refill) * rate)
if tokens >= 1:
    tokens -= 1
    redis.set(key, tokens)
    allow()
else:
    reject()
```

---

## Algorithm 5: Leaky Bucket 🚿

```
Requests enter a queue (bucket)
Queue drains at fixed rate (the "leak")
If queue full → reject new requests
```

- Output is ALWAYS at fixed rate — no bursts
- ✅ Smooths out traffic perfectly
- ✅ Good for protecting downstream services that can't handle spikes
- ❌ Requests queue up — adds latency
- ❌ Burst requests at start of window get queued, not rejected fast

**Difference from Token Bucket:**

- Token Bucket: allows bursts at token bucket capacity
- Leaky Bucket: forces steady rate, queues or drops excess

---

## Where to Implement Rate Limiting

| Layer       | Tool                     | Notes                       |
| ----------- | ------------------------ | --------------------------- |
| Edge/CDN    | Cloudflare Rate Limiting | Best for DDoS protection    |
| API Gateway | AWS API Gateway, Kong    | Easy to configure           |
| Application | Redis-based counter      | Most flexible, custom logic |
| Database    | pg_rate_limit, etc       | Rare, usually too late      |

**In distributed systems:** Use a centralized store (Redis) so all API server instances share the same counter.

---

## Rate Limit Response

```
HTTP 429 Too Many Requests
Headers:
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 0
  X-RateLimit-Reset: 1704067260
  Retry-After: 60
```

---

## ⚖️ Which Algorithm to Choose?

| Scenario                   | Algorithm                            |
| -------------------------- | ------------------------------------ |
| Simple API limits          | Fixed Window (simple to implement)   |
| Accurate sliding limits    | Sliding Window Counter               |
| Allow burst, limit average | Token Bucket ✅ (most common answer) |
| Smooth output rate         | Leaky Bucket                         |

**Default interview answer:** Token Bucket — it's what AWS uses and it handles real-world traffic patterns well.

---

## 🎨 Diagram

See: [`../diagrams/rate-limiter-designs.excalidraw`](../diagrams/rate-limiter-designs.excalidraw)

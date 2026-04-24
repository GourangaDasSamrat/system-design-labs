# 🚦 Rate Limiting

> **One-liner:** Rate limiting controls how many requests a client can make in a given time window — protecting your system from abuse, DDoS, and runaway clients.

---

## 📌 Why Rate Limit?

- **Prevent abuse** — stop scrapers, bots, bad actors
- **Prevent DDoS** — absorb attack traffic before it reaches your servers
- **Fairness** — ensure no single user hogs resources
- **Cost control** — prevent runaway API usage on paid upstream services
- **SLA enforcement** — tier-based limits (free: 100/min, pro: 10K/min)

---

## 🔢 Rate Limiting Algorithms

### 1. Fixed Window Counter

```
Window: 1 minute (60s)
Limit: 100 requests

|------ 0:00 to 1:00 ------|------ 1:00 to 2:00 ------|
  count: 100 ✅                count: 0 (reset)
```

**Implementation:**

```javascript
const key = `rate:${userId}:${Math.floor(Date.now() / 60000)}`;
const count = await redis.incr(key);
await redis.expire(key, 60);
if (count > 100) throw new TooManyRequestsError();
```

✅ Simple, O(1)
❌ **Boundary burst problem:** User sends 100 at 0:59, 100 at 1:01 → 200 in 2 seconds!

---

### 2. Sliding Window Log

Keep a log of timestamps of all recent requests:

```
Requests: [0:30, 0:45, 0:55, 1:05, 1:15]
At time 1:20, window is [0:20 to 1:20]
Valid requests: [0:30, 0:45, 0:55, 1:05, 1:15] = 5
```

```javascript
const now = Date.now();
const windowStart = now - 60000; // 1 minute ago

// Remove old entries, add new one, count
await redis.zremrangebyscore(key, 0, windowStart);
await redis.zadd(key, now, now);
const count = await redis.zcard(key);

if (count > 100) throw new TooManyRequestsError();
```

✅ Accurate, no boundary bursting
❌ Memory-heavy — stores every request timestamp
❌ O(log N) per request

---

### 3. Sliding Window Counter (Best Balance)

Approximate the sliding window using two fixed windows:

```
Current window count: 40 (60% through window)
Previous window count: 80

Estimate: 80 × (1 - 0.60) + 40 = 80 × 0.40 + 40 = 32 + 40 = 72
```

✅ Memory efficient (just 2 counters)
✅ Close to accurate
✅ O(1)

---

### 4. Token Bucket

```
Bucket capacity: 100 tokens
Refill rate: 10 tokens/second

Each request: consume 1 token
No tokens: reject request

Allows bursts up to bucket capacity
```

```python
class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.tokens = capacity
        self.capacity = capacity
        self.refill_rate = refill_rate  # tokens per second
        self.last_refill = time.time()

    def consume(self, tokens=1):
        now = time.time()
        elapsed = now - self.last_refill
        self.tokens = min(self.capacity, self.tokens + elapsed * self.refill_rate)
        self.last_refill = now

        if self.tokens >= tokens:
            self.tokens -= tokens
            return True  # allowed
        return False  # rejected
```

✅ Allows controlled bursts
✅ Smooth rate over time
Used by: AWS API Gateway, Stripe

---

### 5. Leaky Bucket

```
Requests pour in → [Bucket] → drip out at fixed rate
Bucket overflows → requests rejected
```

Unlike token bucket, leaky bucket enforces strict constant rate (no bursts).

✅ Strict rate enforcement
❌ No burst allowance

---

## 📊 Algorithm Comparison

| Algorithm       | Memory | Accuracy             | Burst | Complexity |
| --------------- | ------ | -------------------- | ----- | ---------- |
| Fixed Window    | O(1)   | Low (boundary issue) | Yes   | Simple     |
| Sliding Log     | O(N)   | High                 | No    | Medium     |
| Sliding Counter | O(1)   | Medium               | No    | Simple     |
| Token Bucket    | O(1)   | High                 | Yes   | Medium     |
| Leaky Bucket    | O(1)   | High                 | No    | Medium     |

> **In interviews:** Sliding Window Counter or Token Bucket are the best answers.

---

## 🗺️ Where to Apply Rate Limiting

```
Client → [CDN Rate Limit]       ← block volumetric attacks early
       → [API Gateway Limit]    ← per-user, per-endpoint limits
       → [Service-level Limit]  ← protect individual services
       → [DB Connection Limit]  ← max connections in pool
```

---

## 🔑 Rate Limit Headers

Always return these headers so clients can adapt:

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 63
X-RateLimit-Reset: 1711641600

# When rate limited:
HTTP/1.1 429 Too Many Requests
Retry-After: 30
```

---

## 🏗️ Distributed Rate Limiting

Multiple app servers → each has local counter → no coordination → limits not enforced accurately.

**Solution: Centralized counter in Redis**

```
[App Server 1] ─┐
[App Server 2] ─┼──► [Redis] ← single source of truth for counters
[App Server 3] ─┘
```

Redis INCR is atomic → no race conditions.

---

## 🌍 Rate Limiting in Real Systems

| Company             | Strategy                                        |
| ------------------- | ----------------------------------------------- |
| **GitHub API**      | 5,000 req/hr (authenticated), 60/hr (anonymous) |
| **Twitter API**     | Per-endpoint limits (15 min windows)            |
| **Stripe**          | 100 req/sec per secret key                      |
| **AWS API Gateway** | Token bucket, configurable per stage            |
| **Cloudflare**      | Rule-based at CDN level (before hitting origin) |

---

## 🔑 Key Takeaways

- Rate limiting is a **must-have** in any public API
- Use **Token Bucket** when bursts are acceptable; **Sliding Window Counter** for strict limits
- Always use **Redis** for distributed rate limiting (atomic INCR)
- Return proper headers (`X-RateLimit-*`) so clients can self-throttle
- Apply rate limiting at **multiple layers** (CDN, gateway, service)

---

## 🔗 Related Topics

- [API Gateway](../04-networking-and-routing/api-gateway.md)
- [Redis](../05-caching/redis.md)
- [Load Balancers](../04-networking-and-routing/load-balancers.md)

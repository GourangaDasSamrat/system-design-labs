# ⚡ Circuit Breaker Pattern

> **One-liner:** A circuit breaker stops calling a failing service to give it time to recover — instead of hammering it with requests that are guaranteed to fail.

> 🎨 **Diagram:** [circuit-breaker.excalidraw](../diagrams/circuit-breaker.excalidraw) — open in [Excalidraw](https://excalidraw.com) (File → Open)

---

## ❓ The Problem: Cascading Failures

```
User Request
    ↓
Service A  ──► Service B  ──► Service C (DOWN 💥)
    ↑              ↑
Threads hang   Threads hang
(timeout 30s)  (timeout 30s)

→ Service A's thread pool exhausts
→ Service A goes down
→ Everything upstream dies
→ Full cascade failure 🔥
```

Without circuit breakers, **one slow service kills your entire system**.

---

## 🔌 The Three States

```
┌─────────────────────────────────────────────────────────┐
│                      CLOSED                             │
│              (Normal operation)                         │
│   Requests flow through. Track failure rate.            │
│   Failure threshold exceeded → trip to OPEN             │
└──────────────────────────┬──────────────────────────────┘
                           │ failures > threshold
                           ▼
┌─────────────────────────────────────────────────────────┐
│                       OPEN                              │
│              (Service is DOWN)                          │
│   All requests IMMEDIATELY fail (no network call)       │
│   Return cached/default response                        │
│   Wait for reset timeout (e.g., 60s) → HALF-OPEN       │
└──────────────────────────┬──────────────────────────────┘
                           │ timeout elapsed
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    HALF-OPEN                            │
│              (Testing recovery)                         │
│   Let a few probe requests through                      │
│   Success → CLOSED (recovered! ✅)                      │
│   Failure → back to OPEN (still down ❌)                │
└─────────────────────────────────────────────────────────┘
```

---

## 💻 Implementation Example (Conceptual)

```javascript
class CircuitBreaker {
  constructor(fn, { failureThreshold = 5, resetTimeout = 60000 }) {
    this.fn = fn;
    this.state = "CLOSED";
    this.failureCount = 0;
    this.failureThreshold = failureThreshold;
    this.resetTimeout = resetTimeout;
  }

  async call(...args) {
    if (this.state === "OPEN") {
      // Fail fast — don't even try
      return this.fallback();
    }

    try {
      const result = await this.fn(...args);
      this.onSuccess();
      return result;
    } catch (err) {
      this.onFailure();
      throw err;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    this.state = "CLOSED";
  }

  onFailure() {
    this.failureCount++;
    if (this.failureCount >= this.failureThreshold) {
      this.state = "OPEN";
      setTimeout(() => {
        this.state = "HALF_OPEN";
      }, this.resetTimeout);
    }
  }

  fallback() {
    return { error: "Service temporarily unavailable", cached: true };
  }
}

// Usage:
const paymentBreaker = new CircuitBreaker(callPaymentService, {
  failureThreshold: 5,
  resetTimeout: 30000,
});
```

---

## 🛠️ Production Libraries

| Language     | Library                                |
| ------------ | -------------------------------------- |
| Java         | **Resilience4j**, Hystrix (deprecated) |
| Node.js      | **opossum**                            |
| Go           | **sony/gobreaker**                     |
| Python       | **pybreaker**                          |
| .NET         | **Polly**                              |
| Service mesh | **Istio** (no code changes needed)     |

---

## 🔗 Circuit Breaker + Fallback Strategies

| Fallback Type       | Example                                   |
| ------------------- | ----------------------------------------- |
| **Cached response** | Return last known good data               |
| **Default value**   | Show "0 recommendations" instead of error |
| **Static response** | Return empty array, blank page            |
| **Redirect**        | Send to static maintenance page           |
| **Queue**           | Accept request, process later             |

---

## ✅ Pros

- Prevents cascading failures — protects the whole system
- Fail-fast gives users a quick response instead of 30s timeout
- Gives the failing service breathing room to recover
- Enables graceful degradation with fallbacks

## ❌ Cons

- Adds complexity to every service call
- Threshold tuning is tricky (too sensitive = flapping, too lax = slow to trip)
- Half-open probes can still let some failures through
- Stale cached data shown as fallback may mislead users

## ⚖️ When to Use / When NOT to Use

**✅ Use when:**

- Calling external APIs or microservices (anything that can fail)
- Third-party integrations (payment gateways, SMS providers)
- Inter-service communication in microservices architecture
- Any synchronous call that has a timeout risk

**❌ Avoid when:**

- Calling a local in-process function (no network, no need)
- Using async message queues (they decouple naturally)
- Already using a service mesh like Istio (it handles this for you)

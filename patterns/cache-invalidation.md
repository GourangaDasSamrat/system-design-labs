# 🗑️ Cache Invalidation Patterns

> "There are only two hard things in computer science: cache invalidation and naming things." — Phil Karlton

---

## The Core Problem

Cache = stale data risk. Every caching strategy is a trade-off between:

- **Freshness** — how up-to-date is cached data?
- **Performance** — how often do we go to the DB?
- **Consistency** — does every user see the same thing?

---

## Strategy 1: TTL (Time-To-Live) Expiry

```
cache.set("user:123", userData, ttl=300)  # expires in 5 minutes
```

- Simplest strategy — let cache entries age out
- ✅ No invalidation logic needed
- ❌ Stale window = TTL duration (user sees old data for up to 5 min)
- ❌ Cache stampede on expiry (see below)

**Choose TTL by data freshness requirement:**

- User profile: 5-15 minutes
- Product price: 1-5 minutes
- News article: 1 hour
- Static config: 1 day
- Currency rates: 1 minute

---

## Strategy 2: Write-Through Cache

```
Application writes → Cache AND DB simultaneously
Read → always hits cache (always warm)
```

- ✅ Cache always up-to-date (no stale reads)
- ✅ Reads always fast
- ❌ Write latency doubles (must write both)
- ❌ Cache may fill with data never read (write-heavy workloads waste memory)

**Best for:** Read-heavy workloads where writes are infrequent

---

## Strategy 3: Write-Behind (Write-Back) Cache

```
Application writes → Cache only (async)
Background worker → flushes cache to DB periodically
```

- ✅ Very fast writes (no DB in critical path)
- ❌ Data loss risk if cache crashes before flush
- ❌ Complexity — need background flusher, failure handling

**Best for:** High-write scenarios where some data loss is acceptable (analytics, counters)

---

## Strategy 4: Cache-Aside (Lazy Loading)

```
Read request:
  Check cache → HIT: return ✅
              → MISS: query DB → store in cache → return

Write request:
  Write to DB
  Invalidate (delete) cache entry
```

- Most common pattern
- ✅ Cache only contains data that's actually requested
- ✅ Cache failure doesn't affect writes
- ❌ Cache miss = extra DB round trip (cold start penalty)
- ❌ Race condition: two reads can populate stale data simultaneously

**Race condition fix:** Use a short TTL even when invalidating, or use versioned keys.

---

## Strategy 5: Event-Driven Invalidation

```
DB changes → emit event (via Kafka, SNS, CDC) → cache service → delete/update cache
```

- ✅ Cache invalidated immediately on change
- ✅ Decoupled — cache logic separate from write logic
- ❌ Event delay = brief stale window
- ❌ Event loss = permanently stale cache (need retry/DLQ)

**CDC (Change Data Capture):**

- Debezium reads Postgres WAL → emits change events → Kafka → cache invalidation service
- Low-latency, no application code change required

---

## ⚡ Cache Stampede (Thundering Herd)

When a popular cache key expires → thousands of concurrent requests all hit DB simultaneously → DB overloaded → slow responses → everyone waits → chaos.

**Solutions:**

### Mutex/Lock on Cache Miss

```python
def get_user(id):
    data = cache.get(f"user:{id}")
    if data: return data

    # Acquire lock — only one thread fetches from DB
    with redis.lock(f"lock:user:{id}", timeout=5):
        data = cache.get(f"user:{id}")  # double-check after lock
        if data: return data
        data = db.get_user(id)
        cache.set(f"user:{id}", data, ttl=300)
        return data
```

### Early Expiry / Probabilistic Refresh

- Refresh cache before it actually expires
- Background thread proactively refreshes popular keys at ~80% of TTL

### Stale-While-Revalidate

- Serve stale data immediately
- Async: fetch fresh data in background, update cache
- Users see slightly stale but fast response

---

## Strategy 6: Versioned Cache Keys

Instead of invalidating → use new key:

```
# v1 data at key: user:123:v1
# After update → write to user:123:v2
# Old key auto-expires, new key is fresh
```

- ✅ No invalidation race conditions
- ✅ Old and new data can coexist during rollout
- ❌ Old keys accumulate (need GC / short TTL on old versions)

---

## ⚖️ Summary — Which Pattern When?

| Pattern        | Best For                             |
| -------------- | ------------------------------------ |
| TTL            | Simple cases, infrequent updates     |
| Write-Through  | Read-heavy, must-be-fresh            |
| Write-Behind   | High write throughput, some loss OK  |
| Cache-Aside    | General purpose ← **default choice** |
| Event-driven   | Complex systems, CDC, microservices  |
| Versioned keys | Zero-downtime cache updates          |

**Default interview answer:** Cache-aside + TTL + event-driven invalidation for critical data.

---

## 🎨 Diagram

See: [`../diagrams/cache-invalidation.excalidraw`](../diagrams/cache-invalidation.excalidraw)

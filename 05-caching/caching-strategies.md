# ⚡ Caching Strategies

> **One-liner:** Caching stores the result of expensive operations (DB queries, API calls, computations) so future requests can be served instantly from fast memory instead of recomputing.

---

## 📌 Why Cache?

```
Without cache:
GET /user/42 → DB query (50ms) → response

With cache:
GET /user/42 → Redis lookup (1ms) → response  ← 50x faster!
```

Rule of thumb: **RAM is ~100x faster than SSD, ~1000x faster than network DB calls.**

---

## 🗺️ Caching Layers (From Client to DB)

```
[Browser Cache]          ← HTTP Cache-Control headers
      ↓ miss
[CDN Edge Cache]         ← Cloudflare, CloudFront
      ↓ miss
[Server-side Cache]      ← Redis, Memcached (in-memory)
      ↓ miss
[Database Query Cache]   ← MySQL query cache, Postgres
      ↓ miss
[Database Storage]       ← Actual data on disk
```

---

## 🔄 Cache Reading Strategies

### Cache-Aside (Lazy Loading) — Most Common

```
Read:
1. Check cache → HIT → return data
2. If MISS → query DB → store in cache → return data

Write:
1. Write to DB
2. Invalidate (delete) cache entry
```

```python
def get_user(user_id):
    cached = redis.get(f"user:{user_id}")
    if cached:
        return json.loads(cached)     # cache hit ✅

    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    redis.setex(f"user:{user_id}", 3600, json.dumps(user))  # cache for 1hr
    return user
```

✅ Only caches data that's actually requested
✅ Cache failure is tolerable — fall back to DB
❌ Cache miss penalty: 2 trips (cache + DB)
❌ Stale data possible after writes (until TTL or invalidation)

### Read-Through

```
App → [Cache Layer] → [DB]
       (cache handles the DB lookup on miss)
```

The cache itself fetches from DB on miss. App only talks to cache.

✅ Simplified app code
❌ First request is always slow (cold start)

### Write-Through

```
Write → Cache → DB (synchronously)
```

On every write, update **both** cache and DB synchronously.

✅ Cache always fresh
❌ Higher write latency (must write both)
❌ Writes cache entries that may never be read

### Write-Behind (Write-Back)

```
Write → Cache → ACK to client
                     ↓ (async, later)
                    DB
```

Write to cache first, asynchronously flush to DB.

✅ Very fast writes
❌ Risk of data loss if cache crashes before flush
Use case: Analytics counters, view counts, likes

### Write-Around

```
Write → DB (skip cache)
Read  → Cache → if MISS → DB → store in cache
```

Writes go directly to DB, bypassing cache. Cache is populated on read.

✅ Good for write-once, read-many data
❌ First read after write will miss cache

---

## ⚠️ Cache Eviction Policies

When cache is full, which items to remove?

| Policy                          | How It Works                         | Best For                        |
| ------------------------------- | ------------------------------------ | ------------------------------- |
| **LRU** (Least Recently Used)   | Remove item not accessed for longest | General purpose (Redis default) |
| **LFU** (Least Frequently Used) | Remove least accessed item           | When access frequency matters   |
| **FIFO**                        | Remove oldest inserted item          | Simple use cases                |
| **Random**                      | Remove random item                   | When access pattern is uniform  |
| **TTL**                         | Remove items past their expiry       | Time-sensitive data             |

---

## 🕐 TTL (Time to Live)

Every cache entry should have a TTL — or you risk stale data forever.

| Data Type            | Suggested TTL        |
| -------------------- | -------------------- |
| User session         | 30 minutes (sliding) |
| User profile         | 1–5 minutes          |
| Product details      | 5–30 minutes         |
| Search results       | 1–5 minutes          |
| Homepage content     | 1–10 minutes         |
| Static config/flags  | 1–24 hours           |
| Rarely changing data | Days                 |

---

## 💥 Cache Problems

### Cache Stampede (Thundering Herd)

```
Popular cache key expires at exactly same time
→ 10,000 concurrent requests all miss cache
→ All hit the DB simultaneously
→ DB crashes
```

**Fix:** Probabilistic Early Expiration — randomly refresh before TTL expires:

```python
remaining_ttl = redis.ttl(key)
if remaining_ttl < 30 and random() < 0.1:
    refresh_cache(key)  # Only 10% of requests refresh early
```

Or: **Mutex/Lock** — only one request fetches from DB on miss.

### Cache Penetration

```
Requests for non-existent keys (e.g., userId=-999)
→ Always miss cache
→ Always hit DB
→ DB hammered with useless queries
```

**Fix:** Cache the null result too! `redis.setex("user:-999", 60, "NULL")`

Also: **Bloom Filter** — check if key exists before hitting DB.

### Cache Avalanche

```
Many cache entries expire at the same time
→ Flood of DB queries simultaneously
```

**Fix:** Add jitter (random variation) to TTLs:

```python
ttl = 3600 + random.randint(-300, 300)  # 3600 ± 5 minutes
redis.setex(key, ttl, value)
```

---

## 🔑 Key Takeaways

- Default to **Cache-Aside** for reads — it's the most flexible
- Always set a **TTL** — never cache indefinitely
- Handle **thundering herd, penetration, and avalanche** in production
- Cache only data that changes slowly and is read frequently

---

## 🔗 Related Topics

- [Redis](./redis.md)
- [CDN](../04-networking-and-routing/cdn.md)
- [Read Replicas](../03-databases/read-replicas.md)
- [Bloom Filters](../08-reliability-and-performance/bloom-filters.md)

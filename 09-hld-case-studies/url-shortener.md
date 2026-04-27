# 🔗 HLD: URL Shortener (like bit.ly)

> Design a system that takes a long URL and returns a short URL, and redirects short URLs to their originals.

---

## 1️⃣ Clarify Requirements

### Functional Requirements

- Given a long URL, generate a short URL
- Given a short URL, redirect to the original long URL
- (Optional) Custom aliases: `short.ly/my-brand`
- (Optional) URL expiration
- (Optional) Analytics: click count, referrer, location

### Non-Functional Requirements

- **High availability** — downtime = broken links everywhere
- **Low latency redirects** — < 10ms (cached)
- **Durability** — shortened URLs should work for years
- Reads >> Writes (100:1 read-to-write ratio typical)

---

## 2️⃣ Estimate Scale

```
Write QPS:
  10M URLs shortened per day
  = 10M / 86,400 ≈ 115 writes/sec

Read QPS (100:1 ratio):
  100 × 115 = 11,500 reads/sec
  Peak: ~50,000 reads/sec

Storage:
  Each record: shortURL(7B) + longURL(200B) + metadata(100B) ≈ 307 bytes
  10M URLs/day × 10 years = 36.5B URLs
  36.5B × 307 bytes ≈ 11 TB (very manageable)

Cache:
  80% reads on 20% of URLs (Pareto principle)
  Cache 20% of daily URLs: 10M × 0.2 × 307B ≈ 600 MB/day
```

---

## 3️⃣ API Design

```
# Create short URL
POST /api/shorten
Body: { "longUrl": "https://...", "customAlias": "optional", "expiresAt": "optional" }
Response: { "shortUrl": "https://short.ly/abc1234" }

# Redirect
GET /:shortCode
Response: 301/302 Redirect to longUrl

# Analytics (optional)
GET /api/stats/:shortCode
Response: { clicks: 1000, topCountries: [...] }
```

**301 vs 302:**

- `301 Permanent` → browser caches redirect → fewer server hits → can't update
- `302 Temporary` → browser always checks server → can update/expire → use this for analytics

---

## 4️⃣ Short Code Generation

### Option A: Hash + Truncate

```
MD5(longURL) = "1a79a4d60de6718e8e5b326e338ae533"
Take first 7 chars: "1a79a4d"
shortURL: short.ly/1a79a4d
```

Problem: Collisions — different URLs can produce same 7-char prefix.

### Option B: Base62 Encoding (Recommended)

Character set: `[0-9a-zA-Z]` = 62 characters

```
7 characters × 62^7 = 3.5 trillion unique codes ✅

ID: 123456789
Base62(123456789) = "8M0kX"
shortURL: short.ly/8M0kX
```

Where does the ID come from?

- **Auto-increment DB ID** — simple, but reveals volume/sequence
- **UUID** — random, no collision, but long → truncate carefully
- **Snowflake ID** — distributed, time-sortable, unique (Twitter's approach)

### Option C: Counter Service

A dedicated service generates unique IDs:

```
Counter Service: ID = 1, 2, 3, ... (globally unique)
URL Service: Base62(ID) → short code
```

---

## 5️⃣ High-Level Design

```
                                    [Cache (Redis)]
                                          │
Client ──► [API Gateway + LB] ──► [URL Service] ──► [DB (PostgreSQL)]
                                          │
                                   [Analytics Queue]
                                          │
                                   [Analytics DB]

Redirect flow:
Client ──GET /abc1234──► [URL Service]
                              │
                    ┌─── Redis HIT → 302 Redirect ──► Client
                    │
                    └─── Redis MISS → DB query → cache → 302 Redirect
```

---

## 6️⃣ Database Schema

```sql
CREATE TABLE urls (
  id           BIGSERIAL PRIMARY KEY,
  short_code   VARCHAR(10) UNIQUE NOT NULL,
  long_url     TEXT NOT NULL,
  user_id      BIGINT,
  created_at   TIMESTAMP DEFAULT NOW(),
  expires_at   TIMESTAMP,
  click_count  BIGINT DEFAULT 0
);

CREATE INDEX idx_short_code ON urls(short_code);

-- Analytics table
CREATE TABLE clicks (
  id          BIGSERIAL PRIMARY KEY,
  short_code  VARCHAR(10),
  clicked_at  TIMESTAMP DEFAULT NOW(),
  country     VARCHAR(2),
  referrer    TEXT,
  user_agent  TEXT
);
```

---

## 7️⃣ Caching Strategy

```
Key:   short_code ("abc1234")
Value: long_url
TTL:   24 hours (refresh on hit if needed)

Cache-aside:
1. Check Redis for short_code
2. MISS → query PostgreSQL → store in Redis with TTL
3. HIT → return long_url immediately

Cache hit rate target: >95% (80/20 rule — most traffic on top URLs)
```

---

## 8️⃣ Scalability Deep Dive

### Read Scaling

- Redis cache handles ~95% of redirect traffic
- Read replicas for the remaining DB reads
- CDN for the redirect response itself (if 301)

### Write Scaling

- Writes (new URLs) are much rarer — primary DB + async replication is fine
- If needed: hash short_code → route to shard

### High Availability

- Redis Sentinel or Cluster for cache HA
- DB: Primary + read replicas + automated failover
- Multiple URL service instances behind LB

---

## 9️⃣ Custom Aliases & Expiry

```python
def shorten(long_url, custom_alias=None, expires_at=None):
    if custom_alias:
        if db.exists(custom_alias):
            raise ConflictError("Alias taken")
        short_code = custom_alias
    else:
        id = db.insert(long_url)
        short_code = base62_encode(id)

    db.store(short_code, long_url, expires_at)
    return f"https://short.ly/{short_code}"

def redirect(short_code):
    url_data = cache.get(short_code) or db.get(short_code)
    if not url_data:
        raise NotFoundError()
    if url_data.expires_at and url_data.expires_at < now():
        raise GoneError()  # 410 Gone
    return url_data.long_url
```

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/hld-url-shortener.excalidraw`](../diagrams/hld-url-shortener.excalidraw)

The diagram shows:

- Full system: Client → LB → URL Service → Redis → PostgreSQL
- Shorten flow vs redirect flow (separate paths)
- Cache hit vs miss paths
- Analytics async flow via queue
- DB schema with indexes highlighted

---

## ✅ Trade-offs Summary

| Decision      | Choice                     | Trade-off                                         |
| ------------- | -------------------------- | ------------------------------------------------- |
| 301 vs 302    | 302                        | More server hits, but analytics & expiry work     |
| ID generation | DB auto-increment + Base62 | Simple but sequential — use Snowflake for privacy |
| Cache TTL     | 24hr                       | Stale entries for expired/deleted URLs            |
| SQL vs NoSQL  | PostgreSQL                 | Simple queries, ACID for uniqueness               |

---

## 🔗 Related Topics

- [Caching Strategies](../05-caching/caching-strategies.md)
- [Back-of-Envelope](../01-fundamentals/back-of-envelope.md)
- [Load Balancers](../04-networking-and-routing/load-balancers.md)
- [Interview Framework](../10-interview-prep/framework.md)

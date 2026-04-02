# 📈 Database Scaling

> **One-liner:** Databases are the hardest part to scale — the strategy is: vertical first → read replicas → caching → sharding.

---

## 📌 Scaling Ladder

```
Stage 1: Single DB (< 10K users)
Stage 2: Separate DB server from app server
Stage 3: Add caching (Redis) — reduce DB reads by 80%
Stage 4: Read Replicas — distribute read traffic
Stage 5: Database Sharding — distribute write traffic + data
Stage 6: CQRS + Event Sourcing (advanced)
```

**Never jump to Stage 5 without exhausting Stages 3 and 4.**

---

## 📊 The Read/Write Split

Most web apps: ~90% reads, ~10% writes.

```
Optimize reads first:
  → Add Redis cache (most impactful)
  → Add read replicas
  → Add CDN for static data

Only then optimize writes:
  → Sharding
  → Async writes (queue + batch)
```

---

## 🗃️ Connection Pooling

Never create a new DB connection per request. Use a pool.

```javascript
// pg (Node.js PostgreSQL)
const pool = new Pool({
  max: 20, // max 20 connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

const result = await pool.query("SELECT * FROM users WHERE id = $1", [id]);
```

Without pooling: 1000 req/sec = 1000 new connections = DB crashes.

---

## 🔗 Related Topics

- [Read Replicas](./read-replicas.md)
- [Database Sharding](./sharding.md)
- [Caching Strategies](../05-caching/caching-strategies.md)
- [CAP Theorem](../08-reliability-and-performance/cap-theorem.md)

# 📖 Read Replicas

> **One-liner:** Copies of your primary database that handle read queries, freeing the primary to focus on writes.

---

## 📌 The Problem

Most web applications are **read-heavy** (80-90% reads, 10-20% writes).

A single database server handles both reads and writes → becomes a bottleneck.

```
All traffic → [Primary DB]
              CPU: 95% 🔥
              Connections: maxed out 🔥
```

---

## 💡 The Solution: Read Replicas

Add secondary copies of the database:

- **Primary** (Master) → handles all **writes**
- **Replicas** (Slaves) → handle all **reads**

```
[App Server 1] ──write──► [Primary DB] ──replicates──► [Replica 1]
[App Server 2] ──read──►  [Replica 1]                 [Replica 2]
[App Server 3] ──read──►  [Replica 2]
```

---

## 🔄 How Replication Works

### Synchronous Replication

```
Write → Primary → waits for Replica ACK → confirms to client
```

- ✅ Zero data loss — Replica always has latest data
- ❌ Higher write latency — must wait for replica confirmation

### Asynchronous Replication

```
Write → Primary → confirms to client → replicates to Replica in background
```

- ✅ Lower write latency
- ❌ Replica lag — reads might return slightly stale data
- ❌ If primary crashes before replication, data loss possible

### Semi-Synchronous

```
Write → Primary → waits for ACK from at least 1 replica → confirms
```

- Balance between safety and latency (MySQL's default option)

---

## 📊 Replication Lag

Replica lag is the delay between a write on primary and it being visible on replica.

| Lag        | Impact                                              |
| ---------- | --------------------------------------------------- |
| < 100ms    | Usually fine — imperceptible to users               |
| 100ms – 1s | Acceptable for most use cases                       |
| > 1s       | Noticeable — "I just posted but I can't see it" bug |
| Minutes    | Data consistency issue — investigate urgently       |

### Handling Lag in Code

```javascript
// Read-after-write consistency:
// After a user writes, route their next read to primary

async function getUserProfile(userId, justUpdated = false) {
  if (justUpdated) {
    return await primaryDB.query("SELECT * FROM users WHERE id = ?", [userId]);
  }
  return await replicaDB.query("SELECT * FROM users WHERE id = ?", [userId]);
}
```

---

## 🏗️ Architecture Patterns

### Single Replica (Simple)

```
Writes → [Primary] → [Replica 1] ← Reads
```

### Multiple Replicas (Common)

```
         [Primary] ──► [Replica 1]  \
Writes →      │    ──► [Replica 2]  ──► Read traffic distributed
              └────► [Replica 3]  /
```

### Cascading Replicas

```
[Primary] → [Replica 1] → [Replica 2] → [Replica 3]
```

Reduces load on primary but increases replica lag.

### Regional Replicas (Multi-region)

```
[Primary - Mumbai] ──► [Replica - Singapore]
                   ──► [Replica - US-East]
```

Serve reads from the closest region to the user.

---

## 🔄 Failover: When Primary Dies

1. **Detect failure** — health check fails
2. **Elect new primary** — replica with least lag is promoted
3. **Redirect writes** — app/load balancer routes writes to new primary
4. **Old primary rejoins** — becomes a replica when it comes back

```
Before: App → Primary(A) → Replica(B)
Failure: Primary(A) dies
After:  App → Primary(B)   [B promoted]
              ↑
           Replica(A) [A rejoins as replica]
```

---

## ⚠️ Read Replica Caveats

| Caveat                   | Detail                                               |
| ------------------------ | ---------------------------------------------------- |
| **Eventual consistency** | Reads may return stale data                          |
| **Write bottleneck**     | Replicas help reads; writes still limited to primary |
| **Complex routing**      | Need to distinguish read vs write queries            |
| **Cost**                 | Each replica = another database server               |

---

## 🌍 When to Add Read Replicas

```
DB CPU > 70% sustained → add replica
Read:Write ratio > 5:1 → add replica
Reporting queries slowing down app → add dedicated analytics replica
Multi-region users → add geo-replica
```

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/read-replicas.excalidraw`](../diagrams/read-replicas.excalidraw)

The diagram shows:

- Primary DB receiving writes
- Replication arrows to 2-3 replicas
- App servers routing reads to replicas
- Failover promotion flow

---

## 🔑 Key Takeaways

- Read replicas solve **read scalability** — writes still bottleneck on primary
- Use **async replication** for performance, **sync** for zero data loss
- Always handle **replica lag** in your application code
- Read replicas are the first step; sharding is the next if writes bottleneck

---

## 🔗 Related Topics

- [Database Scaling](./database-scaling.md)
- [Database Sharding](./sharding.md)
- [CAP Theorem](../08-reliability-and-performance/cap-theorem.md)
- [Caching](../05-caching/caching-strategies.md)

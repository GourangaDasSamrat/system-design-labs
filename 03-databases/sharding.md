# 🔀 Database Sharding

> **One-liner:** Splitting a large database into smaller, independent pieces (shards) spread across multiple machines — each shard holds a subset of the data.

---

## 📌 Why Sharding?

Read replicas help with reads. But what when **writes** bottleneck?

```
10M writes/day → 1 Primary DB can't keep up
Data: 100 TB → 1 server can't store it all
```

Sharding splits both storage AND write load.

---

## 💡 How Sharding Works

Imagine a `users` table with 100M rows.

**Without sharding:** All 100M rows on one server.

**With sharding (4 shards):**

```
Shard 0: users where userId % 4 = 0  (25M rows)
Shard 1: users where userId % 4 = 1  (25M rows)
Shard 2: users where userId % 4 = 2  (25M rows)
Shard 3: users where userId % 4 = 3  (25M rows)
```

---

## 🔑 Sharding Strategies

### 1. Hash-Based Sharding

```
shard = hash(shardKey) % numShards
```

✅ Even distribution  
❌ Adding shards → massive rebalancing (use consistent hashing!)  
Best for: User IDs, random keys

### 2. Range-Based Sharding

```
Shard 0: userId 1–1,000,000
Shard 1: userId 1,000,001–2,000,000
...
```

✅ Simple, range queries efficient  
❌ Hot spots if data isn't evenly distributed (new users pile into last shard)  
Best for: Time-series data (date ranges), ordered data

### 3. Directory-Based Sharding

```
[Lookup Service / Mapping Table]
userId 1234 → Shard 3
userId 5678 → Shard 1
```

✅ Most flexible — can move data between shards  
❌ Lookup service becomes bottleneck / SPOF  
Best for: When data movement between shards is needed

### 4. Geographic Sharding

```
Users in India    → Shard-India  (Mumbai)
Users in US       → Shard-US     (Virginia)
Users in Europe   → Shard-EU     (Frankfurt)
```

✅ Low latency for users  
❌ Cross-shard queries for global reports  
Best for: Multi-region products with data residency requirements

---

## 🔑 Choosing a Shard Key

**The most critical decision in sharding.** A bad shard key causes:

- **Hot spots** — all traffic goes to one shard
- **Uneven data** — one shard is 10× larger
- **Cross-shard joins** — expensive queries

### Good Shard Key Properties

| Property           | Why                                          |
| ------------------ | -------------------------------------------- |
| High cardinality   | Many unique values → even distribution       |
| Low correlation    | Shouldn't correlate with access patterns     |
| Immutable          | Changing it means moving data between shards |
| Frequently queried | Queries can be routed to one shard           |

### Common Shard Keys

| System            | Shard Key                    |
| ----------------- | ---------------------------- |
| User data         | `userId`                     |
| Multi-tenant SaaS | `tenantId`                   |
| Messages          | `conversationId` or `userId` |
| Orders            | `customerId` or `orderId`    |
| Time-series       | `timestamp` + `deviceId`     |

---

## ⚠️ Sharding Challenges

### Cross-Shard Queries

```sql
-- This is easy on 1 DB:
SELECT * FROM orders JOIN users ON orders.userId = users.id

-- With sharding: users on Shard 0, orders on Shard 1 → expensive cross-shard join!
```

Solutions:

- Denormalize data (store redundant user info in orders table)
- Application-level joins (fetch from each shard, merge in code)
- Co-locate related data on same shard (order and user on same shard)

### Rebalancing

Adding a new shard → need to move data → expensive operation

Solution: **Consistent hashing** minimizes data movement

### Hot Spots

Celebrity/viral content — 90% of requests go to one shard.

Solution: **Key splitting** — split the hot key across multiple shards with a suffix:

```
tweet:viral_id_0, tweet:viral_id_1, tweet:viral_id_2 ...
```

---

## 🏗️ Sharding Architecture

```
[App Server] → [Shard Router / Proxy Layer]
                    │
          ┌────────┼────────┐
       [Shard 0] [Shard 1] [Shard 2]
          │          │         │
       [Replica] [Replica] [Replica]  (each shard also has replicas!)
```

---

## 🆚 Sharding vs Partitioning

| Term             | Meaning                                                                   |
| ---------------- | ------------------------------------------------------------------------- |
| **Sharding**     | Horizontal partitioning across **different machines**                     |
| **Partitioning** | Splitting data within **one machine** (e.g., PostgreSQL table partitions) |

Start with partitioning (within one DB), then shard when you need cross-machine scale.

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/database-sharding.excalidraw`](../diagrams/database-sharding.excalidraw)

The diagram shows:

- Shard router receiving queries
- Hash-based routing to 3 shards
- Each shard with its own replica
- Cross-shard join problem highlighted in red

---

## 🔑 Key Takeaways

- Sharding splits **both data and write load** across machines
- Shard key choice is **irreversible** — choose carefully
- Start with **vertical scaling → read replicas → sharding** (don't jump to sharding early)
- Every large database (DynamoDB, Cassandra, MongoDB) shards internally for you

---

## 🔗 Related Topics

- [Consistent Hashing](../02-scaling/consistent-hashing.md)
- [Read Replicas](./read-replicas.md)
- [SQL vs NoSQL](./sql-vs-nosql.md)
- [CAP Theorem](../08-reliability-and-performance/cap-theorem.md)

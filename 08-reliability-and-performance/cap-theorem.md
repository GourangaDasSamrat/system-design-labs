# 🔺 CAP Theorem

> **One-liner:** In a distributed system, you can only guarantee two of three properties — Consistency, Availability, and Partition Tolerance — at the same time.

---

## 📌 The Three Properties

### C — Consistency

Every read returns the **most recent write** (or an error). All nodes see the same data at the same time.

```
Node A: Write x=5
Node B: Read x → must return 5 (not an old value)
```

### A — Availability

Every request receives a **non-error response** (but the data might be stale).

```
Node B is out of sync but still responds:
Read x → returns 3 (old value, but NOT an error)
```

### P — Partition Tolerance

The system **continues to operate** even when network partitions cause nodes to be unable to communicate.

```
[Node A] ~~~ NETWORK PARTITION ~~~ [Node B]
System still works (doesn't go down)
```

---

## 🔺 The Triangle

```
         Consistency
             /\
            /  \
           /    \
          / CP   \
         /        \
        /____  ____\
   CA  /      \/     \ AP
      /    PICK 2    \
     /________________\
  Availability      Partition
                    Tolerance
```

**In practice: Partitions happen. You must choose P.**
So the real choice is: **CP** or **AP**.

---

## 🔀 CP Systems — Consistency + Partition Tolerance

When a network partition occurs:

- System **refuses to respond** rather than return stale data
- Prioritizes correctness over availability

```
Node A (Primary) ~~~ partition ~~~ Node B (Replica)

Request to Node B → "I can't reach primary, refusing to serve" → Error/timeout
```

**Examples:** HBase, Zookeeper, etcd, MongoDB (by default config), Google Spanner

**Use when:**

- Banking — wrong balance is worse than no balance
- Inventory — showing wrong stock can cause overselling
- Leader election — must have consistent view of who's the leader

---

## 🔀 AP Systems — Availability + Partition Tolerance

When a network partition occurs:

- System **continues to serve requests** (possibly stale data)
- Prioritizes availability over correctness

```
Node A (Primary) ~~~ partition ~~~ Node B (Replica)

Request to Node B → "I'll serve my stale data" → Responds (maybe stale)
```

**Examples:** Cassandra, DynamoDB, CouchDB, DNS, Riak

**Use when:**

- Social media likes/views — a few seconds lag is fine
- Product catalog — slightly stale price is acceptable
- DNS — serving cached records during failures is fine

---

## 📊 Real Database Classification

| Database   | Type                                     | Notes                                         |
| ---------- | ---------------------------------------- | --------------------------------------------- |
| PostgreSQL | CA (single node) / CP (distributed)      | Single node: no partition                     |
| MySQL      | CA (single node) / CP (with replication) |                                               |
| MongoDB    | CP                                       | Can configure for AP with lower write concern |
| Cassandra  | AP                                       | Tunable consistency (ONE to ALL)              |
| DynamoDB   | AP (default) / CP (with strong reads)    |                                               |
| Redis      | CP (Cluster mode)                        |                                               |
| Zookeeper  | CP                                       | Used for coordination                         |
| HBase      | CP                                       | Strong consistency                            |
| CouchDB    | AP                                       | Conflict resolution                           |

---

## 🔧 Tunable Consistency (Cassandra)

Real systems aren't binary. Cassandra lets you tune per-query:

```
CONSISTENCY ONE   → fastest, least consistent (1 node responds)
CONSISTENCY QUORUM → balanced (majority of nodes respond)
CONSISTENCY ALL   → slowest, most consistent (all nodes respond)
```

**Quorum formula:**

```
Nodes = 5
Quorum = floor(5/2) + 1 = 3

Write to 3 + Read from 3 → at least 1 node overlaps → strong consistency
```

---

## 🔄 PACELC — Extension of CAP

CAP only talks about partition scenarios. **PACELC** extends it:

```
If Partition (P): choose between Availability (A) or Consistency (C)
Else (E):         choose between Latency (L) or Consistency (C)
```

Even without partitions, there's a trade-off:

- Sync replication → strong consistency but **higher latency**
- Async replication → lower latency but **weaker consistency**

| System    | If P | Else |
| --------- | ---- | ---- |
| DynamoDB  | AP   | EL   |
| Cassandra | AP   | EL   |
| MongoDB   | CP   | EC   |
| Spanner   | CP   | EC   |

---

## 🏗️ Eventual Consistency in Practice

AP systems promise **eventual consistency** — given no new writes, all nodes will eventually converge.

```
t=0: Write x=5 to Node A
t=1: Read from Node B → returns 3 (stale)
t=2: Replication happens
t=3: Read from Node B → returns 5 ✅ (converged)
```

**How long does "eventually" take?**

- Same datacenter: milliseconds
- Cross-region: 100ms to seconds
- During partition: until partition heals

---

## 🔑 Key Takeaways

- In practice: **Partition Tolerance is non-negotiable** → choose CP or AP
- **CP** = strong consistency, sacrifice availability during partition
- **AP** = always available, sacrifice consistency (eventual)
- Most modern databases offer **tunable consistency** — you choose per operation
- Match the trade-off to your business need: financial data → CP; social data → AP

---

## 🔗 Related Topics

- [SQL vs NoSQL](../03-databases/sql-vs-nosql.md)
- [Database Sharding](../03-databases/sharding.md)
- [Read Replicas](../03-databases/read-replicas.md)

# 🗄️ SQL vs NoSQL

> **One-liner:** SQL databases store data in structured tables with strict schemas; NoSQL databases trade strict consistency for flexibility and horizontal scalability.

---

## 📌 SQL (Relational Databases)

### Core Properties

- **Structured** — Data in tables with rows and columns
- **Schema** — Fixed structure, must be defined upfront
- **ACID** — Atomicity, Consistency, Isolation, Durability
- **Relations** — Foreign keys, JOINs across tables
- **Query Language** — SQL (standardized)

### Examples

| Database      | Best For                                        |
| ------------- | ----------------------------------------------- |
| PostgreSQL    | Complex queries, JSONB support, general purpose |
| MySQL         | Web apps, e-commerce                            |
| SQLite        | Local/embedded, mobile apps                     |
| Oracle        | Enterprise, banking                             |
| MS SQL Server | Microsoft ecosystem                             |

### When to Use SQL

- ✅ Complex queries with JOINs
- ✅ Strong consistency required (banking, payments)
- ✅ Data is relational by nature (users → orders → products)
- ✅ ACID transactions needed
- ✅ Data structure is well-known and unlikely to change

---

## 📌 NoSQL (Non-Relational Databases)

### Core Properties

- **Flexible schema** — Add/remove fields without migrations
- **Horizontally scalable** — Designed to scale out
- **Eventual consistency** — Usually (some offer strong consistency)
- **Optimized for specific access patterns**

### Types of NoSQL

#### 1. Document Store

```json
// MongoDB, CouchDB, Firestore
{
  "_id": "user_123",
  "name": "Rahul",
  "address": { "city": "Delhi", "pin": "110001" },
  "tags": ["premium", "verified"]
}
```

Best for: User profiles, product catalogs, CMS

#### 2. Key-Value Store

```
Redis, DynamoDB, Memcached
"session:abc123" → { userId: 42, expiry: ... }
"counter:pageviews" → 1000000
```

Best for: Sessions, caching, real-time counters, leaderboards

#### 3. Column-Family Store

```
Cassandra, HBase
Row key → { column1: val, column2: val, column3: val }
```

Best for: Time-series data, IoT, write-heavy workloads, Netflix viewing history

#### 4. Graph Database

```
Neo4j, Amazon Neptune
Nodes (Person) → Edges (FOLLOWS) → Nodes (Person)
```

Best for: Social networks, fraud detection, recommendation engines

---

## ⚖️ The Comparison

| Dimension          | SQL        | NoSQL                      |
| ------------------ | ---------- | -------------------------- |
| Schema             | Fixed      | Flexible                   |
| Horizontal scaling | Hard       | Easy                       |
| Transactions       | Full ACID  | Often eventual consistency |
| JOINs              | Easy       | Avoid (denormalize)        |
| Query flexibility  | High (SQL) | Limited to access patterns |
| Learning curve     | Moderate   | Varies by type             |
| Maturity           | 50+ years  | 15–20 years                |

---

## 🔄 ACID vs BASE

### ACID (SQL)

| Property        | Meaning                                                           |
| --------------- | ----------------------------------------------------------------- |
| **Atomicity**   | All or nothing — transaction either fully succeeds or fully fails |
| **Consistency** | DB always goes from one valid state to another                    |
| **Isolation**   | Concurrent transactions don't interfere                           |
| **Durability**  | Once committed, data survives crashes                             |

### BASE (NoSQL)

| Property                  | Meaning                                        |
| ------------------------- | ---------------------------------------------- |
| **Basically Available**   | System always responds (maybe stale data)      |
| **Soft state**            | State can change over time, even without input |
| **Eventually consistent** | System will eventually be consistent           |

---

## 🏗️ Choosing Between SQL and NoSQL

```
Decision Tree:

Is the data relational?
├── Yes → SQL (start here)
└── No ↓
    Do you need flexible schema?
    ├── Yes → Document (MongoDB)
    └── No ↓
        Is it write-heavy time-series?
        ├── Yes → Column (Cassandra)
        └── No ↓
            Is it graph-traversal heavy?
            ├── Yes → Graph (Neo4j)
            └── No → Key-Value (Redis/DynamoDB)
```

---

## 🌍 Real-World Usage Patterns

| Company       | SQL                    | NoSQL                                |
| ------------- | ---------------------- | ------------------------------------ |
| **Uber**      | PostgreSQL (trips)     | Cassandra (location updates)         |
| **Netflix**   | MySQL (billing)        | Cassandra (viewing history)          |
| **Instagram** | PostgreSQL (user data) | Redis (feed cache)                   |
| **Twitter**   | MySQL                  | Redis (timeline), Cassandra (tweets) |
| **Airbnb**    | MySQL                  | ElasticSearch (search)               |

> Most large systems use **both** — polyglot persistence!

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/sql-vs-nosql.excalidraw`](../diagrams/sql-vs-nosql.excalidraw)

---

## 🔑 Key Takeaways

- **Default to SQL** for new projects — better tooling, stronger guarantees
- **Move to NoSQL** when you hit specific scale/flexibility walls
- Most production systems use **both** (polyglot persistence)
- NoSQL doesn't mean "no schema" — you still have implicit structure

---

## 🔗 Related Topics

- [Database Scaling](./database-scaling.md)
- [Read Replicas](./read-replicas.md)
- [Database Sharding](./sharding.md)
- [CAP Theorem](../08-reliability-and-performance/cap-theorem.md)

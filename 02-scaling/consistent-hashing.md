# 🔄 Consistent Hashing

> **One-liner:** A hashing technique that minimizes the number of keys that need to be remapped when servers are added or removed.

---

## 📌 The Problem It Solves

Imagine you have 3 cache servers and you distribute keys using:

```
server = hash(key) % N  (N = number of servers)
```

**Works fine until N changes:**

```
N=3: hash("user123") % 3 = 1 → Server 1
N=4: hash("user123") % 4 = 3 → Server 3  ← DIFFERENT!
```

When you add or remove a server, **almost ALL keys remap** → massive cache miss storm → database gets hammered.

---

## 💡 The Solution: Consistent Hashing

### How It Works

1. Map both **servers** and **keys** onto the same circular hash ring (0 to 2³²)
2. Each key is assigned to the **first server clockwise** from its position
3. Adding/removing a server only affects **adjacent keys**

```
         0
      /     \
  S3         S1
    \       /
         S2
        360°

Key K1 → clockwise → lands on S1
Key K2 → clockwise → lands on S2
```

### Adding a Server

```
Before:  K1→S1, K2→S1, K3→S2, K4→S3
Add S4 between S1 and S2:
After:   K1→S1, K2→S4, K3→S2, K4→S3  ← Only K2 moved!
```

### Removing a Server

```
Before: K1→S1, K2→S2, K3→S2, K4→S3
Remove S2:
After:  K1→S1, K2→S3, K3→S3, K4→S3  ← K2 and K3 moved to next server
```

---

## ⚖️ Problem: Uneven Distribution

With few servers, the ring can be unbalanced:

```
S1 handles 60% of keyspace
S2 handles 30% of keyspace
S3 handles 10% of keyspace
```

### Solution: Virtual Nodes (VNodes)

Instead of 1 point per server, use **many virtual points**:

```
S1 → S1_1, S1_2, S1_3 ... S1_100  (spread around ring)
S2 → S2_1, S2_2, S2_3 ... S2_100
S3 → S3_1, S3_2, S3_3 ... S3_100
```

Result: Much more even distribution. Each physical server maps to ~N/total keys.

---

## 🏗️ Implementation Sketch

```javascript
class ConsistentHash {
  constructor(replicas = 100) {
    this.replicas = replicas; // virtual nodes per server
    this.ring = new Map(); // hash → serverName
    this.sortedKeys = [];
  }

  addServer(server) {
    for (let i = 0; i < this.replicas; i++) {
      const hash = md5(`${server}:${i}`);
      this.ring.set(hash, server);
      this.sortedKeys.push(hash);
    }
    this.sortedKeys.sort();
  }

  getServer(key) {
    const hash = md5(key);
    // Find first server clockwise
    for (const k of this.sortedKeys) {
      if (k >= hash) return this.ring.get(k);
    }
    return this.ring.get(this.sortedKeys[0]); // wrap around
  }
}
```

---

## 📊 Consistent Hashing vs Modulo Hashing

| Dimension               | Modulo Hashing       | Consistent Hashing         |
| ----------------------- | -------------------- | -------------------------- |
| Keys remapped on resize | ~100%                | ~1/N                       |
| Uneven distribution     | Even                 | Possible (fix with VNodes) |
| Complexity              | O(1)                 | O(log N) per lookup        |
| Use case                | Fixed set of servers | Dynamic server pool        |

---

## 🌍 Real-World Use Cases

| System                      | How it's used                  |
| --------------------------- | ------------------------------ |
| **Cassandra**               | Partitions data across nodes   |
| **DynamoDB**                | Key-based partitioning         |
| **Memcached/Redis Cluster** | Distributing cache keys        |
| **Nginx**                   | Consistent upstream selection  |
| **CDN**                     | Routing requests to edge nodes |
| **Kafka**                   | Partition assignment           |

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/consistent-hashing.excalidraw`](../diagrams/consistent-hashing.excalidraw)

The diagram shows:

- The hash ring as a circle
- Server nodes placed on the ring
- Keys mapped to nearest server clockwise
- Virtual nodes distributed around the ring
- Before/after adding a server (minimal key movement)

---

## 🔑 Key Takeaways

- Modulo hashing causes thundering herd on any server change
- Consistent hashing limits disruption to ~1/N keys per change
- Virtual nodes (VNodes) solve the uneven distribution problem
- This is used **everywhere** in distributed databases and caches

---

## 🔗 Related Topics

- [Horizontal Scaling](./horizontal-scaling.md)
- [Database Sharding](../03-databases/sharding.md)
- [Redis](../05-caching/redis.md)

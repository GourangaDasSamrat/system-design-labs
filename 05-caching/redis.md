# 🔴 Redis

> **One-liner:** An in-memory data structure store used as a cache, message broker, session store, rate limiter, and much more — blazing fast because everything lives in RAM.

---

## 📌 Why Redis?

- **Speed:** In-memory → sub-millisecond latency (< 1ms)
- **Rich data types:** Not just key-value — lists, sets, hashes, sorted sets, streams
- **Persistence:** Optional — can snapshot to disk (RDB) or log every write (AOF)
- **Pub/Sub:** Built-in message broker
- **Expiry:** TTL on any key
- **Atomic operations:** All commands are atomic

---

## 🗃️ Redis Data Types

### String — The Basic Type

```redis
SET user:42:name "Rahul"
GET user:42:name      → "Rahul"
INCR page:views       → 1, 2, 3 (atomic counter)
SETEX session:abc 3600 "userId=42"  (with TTL)
```

### Hash — Object/Map

```redis
HSET user:42 name "Rahul" age 25 city "Delhi"
HGET user:42 name      → "Rahul"
HGETALL user:42        → { name: Rahul, age: 25, city: Delhi }
```

Best for: User profiles, product data, config

### List — Ordered, Allows Duplicates

```redis
LPUSH notifications:42 "You got a like!"  (push to left)
RPUSH notifications:42 "New follower!"    (push to right)
LRANGE notifications:42 0 9               (get first 10)
LPOP notifications:42                     (remove from left)
```

Best for: Activity feeds, queues (FIFO/LIFO), recent items

### Set — Unordered, Unique Values

```redis
SADD followers:42 101 102 103
SMEMBERS followers:42    → {101, 102, 103}
SISMEMBER followers:42 102 → 1 (yes)
SCARD followers:42          → 3 (count)
SINTER followers:42 followers:99  → mutual followers (intersection)
```

Best for: Tags, unique visitors, friend lists

### Sorted Set (ZSet) — Ranked/Scored Set

```redis
ZADD leaderboard 9500 "player_alice"
ZADD leaderboard 8700 "player_bob"
ZADD leaderboard 9900 "player_charlie"
ZRANGE leaderboard 0 2 WITHSCORES REV  → charlie(9900), alice(9500), bob(8700)
ZRANK leaderboard "player_alice"         → 1 (0-indexed rank)
```

Best for: **Leaderboards**, rate limiting, priority queues, trending topics

### Stream — Append-only Log

```redis
XADD events * userId 42 action "purchase" amount 999
XREAD COUNT 10 STREAMS events 0
```

Best for: Event sourcing, activity logs, message queues (like Kafka-lite)

---

## 🏗️ Common Redis Use Cases

### 1. Session Store

```javascript
// Login
redis.setex(`session:${token}`, 3600, JSON.stringify({ userId: 42 }));

// Per-request auth
const session = redis.get(`session:${token}`);
```

### 2. Cache (Cache-Aside)

```javascript
const cached = await redis.get(`user:${userId}`);
if (!cached) {
  const user = await db.findUser(userId);
  await redis.setex(`user:${userId}`, 300, JSON.stringify(user));
}
```

### 3. Rate Limiting (Sliding Window)

```javascript
const key = `rate:${userId}:${Math.floor(Date.now() / 60000)}`;
const count = await redis.incr(key);
await redis.expire(key, 60);
if (count > 100) throw new Error("Rate limit exceeded");
```

### 4. Pub/Sub Messaging

```javascript
// Publisher
redis.publish(
  "notifications",
  JSON.stringify({ userId: 42, msg: "New order!" }),
);

// Subscriber
redis.subscribe("notifications", (message) => {
  const { userId, msg } = JSON.parse(message);
  sendPushNotification(userId, msg);
});
```

### 5. Distributed Lock

```javascript
const lock = await redis.set("lock:resource", "1", "NX", "EX", 10);
if (!lock) throw new Error("Resource locked by another process");
// ... do work ...
redis.del("lock:resource");
```

`NX` = set only if Not eXists | `EX 10` = expire in 10 seconds

### 6. Leaderboard

```javascript
// Add/update score
redis.zadd("game:leaderboard", score, userId);

// Top 10
redis.zrange("game:leaderboard", 0, 9, "REV", "WITHSCORES");

// User's rank
redis.zrevrank("game:leaderboard", userId);
```

---

## 🔄 Redis Persistence

| Mode                       | How                       | Data Safety                | Performance |
| -------------------------- | ------------------------- | -------------------------- | ----------- |
| **No persistence**         | Pure in-memory            | Data lost on restart       | Fastest     |
| **RDB (Snapshot)**         | Periodic snapshot to disk | Up to minutes of data loss | Fast        |
| **AOF (Append Only File)** | Log every write to disk   | Nearly no data loss        | Slower      |
| **RDB + AOF**              | Both                      | Best safety                | Moderate    |

```redis
# redis.conf
save 900 1      # Save if 1 key changed in 900 seconds
save 300 10     # Save if 10 keys changed in 300 seconds
appendonly yes  # Enable AOF
```

---

## 🏗️ Redis Cluster vs Sentinel

### Redis Sentinel (High Availability)

```
[Sentinel 1] [Sentinel 2] [Sentinel 3]  ← monitors
      │
[Primary] ──replicates──► [Replica 1]
                         [Replica 2]
```

- Automatic failover — promotes replica to primary
- No sharding — all data on primary
- Use for: HA without massive scale

### Redis Cluster (Scale + HA)

```
[Node 1: slots 0-5460]     [Replica 1a] [Replica 1b]
[Node 2: slots 5461-10922] [Replica 2a] [Replica 2b]
[Node 3: slots 10923-16383][Replica 3a] [Replica 3b]
```

- 16,384 hash slots distributed across nodes
- Built-in sharding + HA
- Use for: Data larger than single machine's RAM

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/redis.excalidraw`](../diagrams/redis.excalidraw)

The diagram shows:

- Redis data types illustrated with examples
- Cache-aside pattern
- Redis Sentinel failover
- Redis Cluster slot distribution

---

## 🔑 Key Takeaways

- Redis is not just a cache — it's a **data structure server**
- Sorted Sets for leaderboards/rankings are a killer feature
- Always set TTLs — unbounded memory growth will kill your Redis
- Use Sentinel for HA, Cluster for scale beyond single-machine RAM

---

## 🔗 Related Topics

- [Caching Strategies](./caching-strategies.md)
- [Rate Limiting](../08-reliability-and-performance/rate-limiting.md)
- [Message Queues](../06-async-communication/message-queues-sqs.md)
- [Pub-Sub](../06-async-communication/pub-sub-sns.md)

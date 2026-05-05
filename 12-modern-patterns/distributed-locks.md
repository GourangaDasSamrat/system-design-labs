# 🔐 Distributed Locks

> **One-liner:** A distributed lock ensures that only one node in a cluster can perform a critical operation at a time — preventing race conditions across services.

> 🎨 **Diagram:** [distributed-locks.excalidraw](../diagrams/distributed-locks.excalidraw) — open in [Excalidraw](https://excalidraw.com) (File → Open)

---

## ❓ Why Do You Need Distributed Locks?

In a single-server world, a mutex or semaphore handles concurrency. But in distributed systems:

- Multiple service replicas run simultaneously
- They all share the same database or resource
- Without coordination, two instances might process the same job, double-charge a user, or corrupt shared state

**Classic example:** Flash sale — 100 units in stock, 10,000 concurrent requests. Without a lock, you oversell.

---

## 🔑 Redis Distributed Lock (Redlock)

The most common approach: use Redis `SET NX EX`.

```bash
# Acquire lock
SET lock:resource_id <unique_token> NX EX 10
# NX = only set if key does NOT exist
# EX 10 = auto-expire in 10 seconds (TTL safety net)

# Returns OK → you have the lock
# Returns nil → someone else has it

# Release lock (Lua script — atomic!)
if redis.call("GET", KEYS[1]) == ARGV[1] then
  return redis.call("DEL", KEYS[1])
else
  return 0
end
```

**Why a unique token?** Prevents a slow process from releasing a lock that was already acquired by someone else after TTL expiry.

---

## ⚙️ How It Works (Step by Step)

```
Service A                    Redis                   Service B
   |                           |                        |
   |── SET lock:job1 tokenA ──>|                        |
   |   NX EX 10                |                        |
   |<── OK (lock acquired) ────|                        |
   |                           |                        |
   |   (doing work...)         |── SET lock:job1 ──────>|
   |                           |   tokenB NX EX 10      |
   |                           |<── nil (locked) ───────|
   |                           |   (retry or fail)      |
   |                           |                        |
   |── DEL lock:job1 ─────────>|                        |
   |   (if token matches)      |                        |
   |<── lock released ─────────|                        |
```

---

## ⚠️ Edge Cases & Gotchas

| Problem                  | What Happens                          | Solution                        |
| ------------------------ | ------------------------------------- | ------------------------------- |
| Process crashes          | Lock stuck forever                    | TTL auto-expire                 |
| Network partition        | Two leaders think they have lock      | Redlock (multi-node)            |
| GC pause longer than TTL | Process resumes, lock already expired | Fencing tokens                  |
| Redis crashes            | Lock lost                             | Use Redis Cluster / persistence |

### Fencing Token Pattern

Each lock acquisition returns a monotonically increasing token number. The resource (e.g., DB) only accepts writes with tokens higher than the last seen:

```
Lock acquired → token 42
Lock expired, re-acquired → token 43
Stale process tries to write with token 42 → REJECTED
```

---

## 🏛️ Alternatives to Redis Locks

| Tool               | Mechanism                  | Use When                     |
| ------------------ | -------------------------- | ---------------------------- |
| **Redis (SET NX)** | Atomic key-value           | Most common, low latency     |
| **ZooKeeper**      | Ephemeral nodes + watchers | Need strong consistency      |
| **etcd**           | Leases + CAS               | Kubernetes-style infra       |
| **DB row lock**    | `SELECT FOR UPDATE`        | Already using SQL, low scale |

---

## ✅ Pros

- Sub-millisecond lock acquisition with Redis
- TTL prevents deadlocks from crashed processes
- Simple to implement with `SET NX EX`
- Scales with Redis Cluster

## ❌ Cons

- Redis is single point of failure (use Redlock for HA)
- Clock drift can cause issues with TTL-based expiry
- Redlock is controversial (Martin Kleppmann vs. Antirez debate)
- Not suitable for long-held locks (use queue instead)

## ⚖️ When to Use / When NOT to Use

**✅ Use when:**

- Preventing double processing (payment, job scheduling)
- Flash sales / inventory deduction
- Leader election among service instances
- Rate limiting per resource

**❌ Avoid when:**

- Operations take longer than a reasonable TTL (use idempotency + queue instead)
- You need guaranteed strong consistency (use ZooKeeper + fencing)
- The resource is already serialized (single-threaded queue consumer)

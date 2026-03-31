# ↕️ Vertical Scaling (Scale Up)

> **One-liner:** Make the existing machine bigger and more powerful.

## What Is It?

Add more **CPU, RAM, SSD** to your existing server — no architecture changes needed.

## Example

```
Before:  4 CPU cores, 16 GB RAM, 500 GB SSD
After:  32 CPU cores, 256 GB RAM, 4 TB NVMe SSD
```

## ✅ Pros

- No code changes needed
- No distributed systems complexity
- Works great up to mid-scale
- Simple operations (just resize the VM)

## ❌ Cons

- **Hard limit** — you can't scale beyond the world's biggest machine
- **Expensive** — high-end servers cost exponentially more
- **Single Point of Failure** — if this one server goes down, everything dies
- **Downtime during upgrades** — usually requires a restart

## When To Use

- Early-stage startup with <100K users
- Databases that are hard to distribute (start vertical, then add read replicas)
- Quick fix while you architect horizontal scaling

---

# ↔️ Horizontal Scaling (Scale Out)

> **One-liner:** Add more machines to spread the load.

## What Is It?

Instead of one big machine, you run **many smaller, identical machines** behind a load balancer.

## Example

```
Before:  1 server handling 1,000 req/sec
After:  10 servers each handling 100 req/sec
```

## ✅ Pros

- **No ceiling** — can add machines indefinitely (theoretically)
- **No SPOF** — if one server dies, others take over
- **Cost-efficient** — commodity hardware is cheap
- **Geographic distribution** — servers in multiple regions

## ❌ Cons

- **Stateless servers required** — session state can't live on one machine
- **Distributed systems complexity** — data consistency, network partitions
- **Load balancer required** — new component to manage
- **Shared resources** — DB becomes a bottleneck (need to solve separately)

## Making Servers Stateless

To scale horizontally, your app servers **must be stateless**:

```
❌ Bad: Session stored in memory on Server A
       → User hits Server B next request → session lost

✅ Good: Session stored in Redis (shared)
        → Any server can serve any request
```

---

## 📊 Comparison Table

| Dimension          | Vertical    | Horizontal             |
| ------------------ | ----------- | ---------------------- |
| Complexity         | Low         | High                   |
| Cost curve         | Exponential | Linear                 |
| Failure impact     | Total       | Partial                |
| Max scale          | Limited     | Near-infinite          |
| Code changes       | None        | Statelessness required |
| Speed to implement | Fast        | Slower                 |

---

## 🏗️ Architecture: Scaling Journey

```
Stage 1 (1K users):
[Client] → [Single Server + DB]

Stage 2 (10K users):
[Client] → [App Server] → [Separate DB Server]

Stage 3 (100K users):
[Client] → [Load Balancer] → [App Server 1]  → [Primary DB]
                           → [App Server 2]  ↘ [Read Replica]
                           → [App Server 3]  ↗

Stage 4 (1M users):
+ CDN, Caching Layer (Redis), Multiple DB shards
```

---

## 🔑 Key Takeaways

- **Start vertical** — it's simpler and fine for early stage
- **Plan for horizontal** — architect for statelessness from day 1
- The real challenge of horizontal scaling is the **database**, not app servers
- Most large systems use **both** — vertically scaled DB with horizontally scaled app tier

---

## 🔗 Related Topics

- [Load Balancers](../04-networking-and-routing/load-balancers.md)
- [Read Replicas](../03-databases/read-replicas.md)
- [Consistent Hashing](./consistent-hashing.md)
- [Caching](../05-caching/caching-strategies.md)

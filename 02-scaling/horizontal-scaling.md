# ↔️ Horizontal Scaling

> See [Vertical Scaling](./vertical-scaling.md) for the full comparison.

**Quick summary:** Add more machines instead of making one machine bigger.

```
[Load Balancer]
    ├── [Server 1]  ← handles 1/N traffic
    ├── [Server 2]  ← handles 1/N traffic
    └── [Server N]  ← handles 1/N traffic
```

**Key requirement:** Servers must be **stateless** — store sessions in Redis, not in-memory.

---

## 🔗 Related Topics

- [Vertical Scaling](./vertical-scaling.md) ← full notes + diagram here
- [Consistent Hashing](./consistent-hashing.md)
- [Load Balancers](../04-networking-and-routing/load-balancers.md)

# 🌍 Multi-Region & Geo-Distribution

> Senior-level signal 💯 — most candidates skip this entirely. Knowing this separates you.

---

## Why Multi-Region?

| Reason                | Explanation                                                  |
| --------------------- | ------------------------------------------------------------ |
| **Latency**           | Serve users from the nearest region (100ms vs 400ms)         |
| **Availability**      | One region down ≠ whole system down                          |
| **Compliance**        | GDPR, data residency laws (EU data must stay in EU)          |
| **Disaster Recovery** | Survive regional cloud outages (AWS us-east-1 has gone down) |

---

## 🗺️ Deployment Models

### Active-Passive

- One region handles ALL traffic
- Passive region is on standby (warm or cold)
- Failover takes time (seconds to minutes)
- ✅ Simpler, cheaper
- ❌ Wasted capacity, failover downtime

### Active-Active

- Multiple regions ALL handle traffic simultaneously
- Load balanced across regions
- ✅ Full utilization, instant failover
- ❌ Data consistency is HARD (writes to two regions = conflict risk)

---

## 🔄 Data Replication Across Regions

### Synchronous Replication

- Write confirms only after all replicas ACK
- ✅ Zero data loss
- ❌ High latency (write waits for far region)

### Asynchronous Replication

- Write confirms after primary writes; replicas catch up
- ✅ Low write latency
- ❌ Replication lag — RPO > 0 (can lose recent writes on failover)

### Conflict Resolution (Active-Active)

- **Last Write Wins (LWW)** — timestamp determines winner (risk: clock skew)
- **Vector Clocks** — track causality, not just time
- **CRDTs** — data structures that merge automatically (counters, sets)
- **Application-level** — humans or business logic resolves conflicts

---

## 🧭 Latency-Based Routing

Route users to the nearest healthy region:

```
User in London → EU-WEST region
User in Singapore → AP-SOUTHEAST region
User in NYC → US-EAST region
```

**How it works:**

- DNS-level routing (Route 53 latency routing, Cloudflare)
- Measures latency from user to each region endpoint
- Routes to lowest-latency region
- Falls back to next region if primary is unhealthy

**Other routing strategies:**

- **Geolocation routing** — by country/continent (compliance use case)
- **Weighted routing** — 90% to main region, 10% to new region (blue/green)
- **Failover routing** — primary + health check → failover on failure

---

## 💥 Disaster Recovery Concepts

| Term         | Definition                                             |
| ------------ | ------------------------------------------------------ |
| **RTO**      | Recovery Time Objective — how long can system be down? |
| **RPO**      | Recovery Point Objective — how much data can we lose?  |
| **Failover** | Switch traffic to backup region                        |
| **Failback** | Return to primary after recovery                       |
| **Runbook**  | Step-by-step incident playbook                         |

### DR Tiers

| Tier         | RTO     | RPO     | Cost | Strategy                       |
| ------------ | ------- | ------- | ---- | ------------------------------ |
| Cold Standby | Hours   | Hours   | $    | Data backed up, infra off      |
| Warm Standby | Minutes | Minutes | $$   | Infra running, minimal traffic |
| Hot Standby  | Seconds | Seconds | $$$  | Full mirror, active-active     |

---

## ⚖️ Trade-offs

✅ **Pros**

- Sub-100ms latency for global users
- Survives regional cloud outages
- Compliance with data residency laws

❌ **Cons**

- Data consistency is fundamentally harder
- Cost multiplier (2x–3x infrastructure)
- Operational complexity (multi-region debugging, data sync issues)

⚖️ **When to use**

- Global user base (users in 3+ continents)
- SLAs requiring 99.99%+ uptime
- Revenue impact of downtime > cost of multi-region

**When NOT to use**

- Small startup with users in one country
- Early stage — single-region with good backups is fine

---

## 🎨 Diagram

See: [`diagrams/multi-region.excalidraw`](../diagrams/multi-region.excalidraw)

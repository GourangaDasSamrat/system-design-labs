# 💰 Cost Optimization

> Most candidates ignore this — huge differentiator 🔥 Interviewers love candidates who think about cost.

---

## The Mindset: Performance vs Cost

Every design decision has a cost. Good engineers think in **cost per unit of work**.

```
Don't just ask: "What's the most scalable solution?"
Ask:            "What's the most scalable solution FOR THIS BUDGET?"
```

---

## 🧮 Cost vs Performance Trade-offs

### Caching vs Compute

| Approach                   | Cost                       | Latency | Freshness             |
| -------------------------- | -------------------------- | ------- | --------------------- |
| Compute every request      | High CPU cost              | Higher  | Always fresh          |
| Cache in Redis (in-memory) | Memory cost (~$0.01/GB-hr) | ~1ms    | Stale by TTL          |
| Cache at CDN edge          | Bandwidth cost             | ~5ms    | Stale by TTL          |
| Pre-compute & store        | Storage cost               | ~1ms    | Stale until recompute |

**Rule of thumb:** If the same data is read 100x more than it's written → cache it.

### Read Replicas vs Scaling Primary

- Adding read replicas: cheaper than scaling the primary
- Primary is the bottleneck for writes; replicas handle reads
- Cost: 1 primary ($200/mo) + 2 replicas ($100/mo each) vs 1 massive primary ($600/mo)

---

## 📦 Storage Tiering

Not all data needs to be fast. Match storage speed to access frequency:

| Tier    | Technology           | Cost | Access Time   | Use Case                         |
| ------- | -------------------- | ---- | ------------- | -------------------------------- |
| Hot     | SSD / NVMe           | $$$  | < 1ms         | Active user data, recent records |
| Warm    | HDD / Standard S3    | $$   | 10–100ms      | Last 90 days of logs, old orders |
| Cold    | S3-IA, Glacier       | $    | Minutes–hours | Compliance archives, old backups |
| Archive | Glacier Deep Archive | ¢    | 12–48 hours   | Legal hold, never-accessed data  |

**Interview tip:** Propose tiering when the interviewer mentions "we have 10 years of data." Storing all of it on SSD is wasteful — tier it.

---

## ⚙️ Instance Right-Sizing

- **Over-provisioning** = money wasted on idle CPU/RAM
- **Under-provisioning** = throttling, poor UX
- Tools: AWS Cost Explorer, GCP Recommender, CloudWatch metrics

**Spot / Preemptible Instances**

- Up to 90% cheaper than on-demand
- Can be reclaimed by cloud provider with 2-minute notice
- ✅ Use for: batch jobs, ML training, stateless workers
- ❌ Don't use for: databases, stateful services, API servers

---

## 🗄️ Database Cost Tricks

| Trick                                                | Saving                                 |
| ---------------------------------------------------- | -------------------------------------- |
| Use read replicas for analytics queries              | Don't tax primary                      |
| Partition old data to cheap storage                  | $$$→$ for cold rows                    |
| Use DynamoDB On-Demand for spiky, low-volume traffic | Pay per request                        |
| Use Aurora Serverless for dev/staging DBs            | 0 cost when idle                       |
| Index correctly                                      | Avoid full table scans = less I/O cost |

---

## 🌐 CDN vs Origin Cost

- **Without CDN:** Every request hits your servers (compute + bandwidth cost)
- **With CDN:** Cache hit ratio of 80%+ means 80% less origin load
- CDN bandwidth: ~$0.01/GB vs EC2 egress: ~$0.09/GB → 9x cheaper

---

## 🔁 Async Processing to Cut Peak Compute

- Synchronous: request waits → need enough servers for PEAK traffic
- Async + queue: requests enqueue → process at steady rate → smaller fleet
- Example: Image processing. 1000 uploads/minute at peak.
  - Sync: need 1000 workers provisioned at all times
  - Async + SQS: 50 workers process queue, backlog drains within minutes

---

## ⚖️ Trade-offs

✅ **Pros of cost optimization**

- Lower burn rate (critical at startups)
- Forces engineering discipline
- Scales better — inefficiencies compound at scale

❌ **Cons / Risks**

- Over-optimizing early wastes engineering time
- Spot instances add operational complexity
- Storage tiering adds retrieval latency

⚖️ **When to bring this up in interviews**

- When asked about scaling to millions of users
- When interviewer asks "what else would you consider?"
- When discussing database choices (cost of managed vs self-hosted)

---

## 🎨 Diagram

See: [`diagrams/cost-optimization.excalidraw`](../diagrams/cost-optimization.excalidraw)

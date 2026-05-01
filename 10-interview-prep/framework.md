# System Design Interview Framework

> Clarify → Estimate → Design → Deep Dive → Trade-offs

---

## Time Management (45-minute interview)

| Phase                | Time   | What To Do                   |
| -------------------- | ------ | ---------------------------- |
| Clarify Requirements | 5 min  | Ask questions, nail scope    |
| Estimate Scale       | 5 min  | DAU, QPS, storage            |
| High-Level Design    | 15 min | Core components, data flow   |
| Deep Dive            | 15 min | 2-3 components in detail     |
| Trade-offs & Wrap-up | 5 min  | Justify choices, limitations |

---

## Phase 1: Clarify Requirements (5 min)

Never assume. Always ask.

### Functional Questions

- "What are the core features I should design for?"
- "Should I include [feature X] or is that out of scope?"
- "What actions can a user take?"
- "Is real-time required, or eventual consistency is fine?"

### Non-Functional Questions

- "How many DAU are we targeting? 1M, 10M, 100M?"
- "What's the read-to-write ratio?"
- "What's the acceptable latency? < 100ms? < 1s?"
- "Do we need global distribution, or is one region fine?"
- "What's the durability requirement? Can we lose data?"

### Assumptions to State Out Loud

- "I'll assume 10M DAU to start"
- "I'll focus on the core read path first"
- "I'll skip auth for now unless you want me to cover it"

---

## Phase 2: Estimate Scale (5 min)

Show your math. Interviewers care about the process.

### Template

```
Users:
  Daily Active Users (DAU): X
  Monthly Active Users (MAU): Y

Traffic:
  Avg requests per user per day: N
  Total requests/day: X × N
  Read QPS: (X × N) / 86,400
  Write QPS: (read QPS) / (read:write ratio)
  Peak QPS: average × 2-3

Storage:
  Record size: (field sizes added up)
  New records/day: (write QPS × 86,400)
  Annual growth: (daily × 365)
  5-year storage: (annual × 5)

Bandwidth:
  Outbound: read QPS × avg response size
```

---

## Phase 2b: API Design (part of HLD)

Design the core API endpoints before diving into architecture. Interviewers notice when you skip this.

### Template

```
Method  Endpoint                 Description
POST    /api/v1/tweets           Create a tweet
GET     /api/v1/tweets/{id}      Get a tweet
GET     /api/v1/feed             Get home timeline
POST    /api/v1/follow/{userId}  Follow a user
DELETE  /api/v1/follow/{userId}  Unfollow

Request body:  { field: type, ... }
Response:      { id, data, metadata, cursor }
Auth:          Bearer JWT in Authorization header
Versioning:    /v1/ prefix — plan for /v2/ if schema may change
```

### Common API Decisions to State Out Loud

- REST vs gRPC (external vs internal)
- Pagination: cursor-based or offset-based?
- Rate limits per endpoint (stricter on writes)
- Idempotency keys for non-idempotent mutations

---

## Phase 2c: DB Schema (part of HLD)

Sketch the core tables/collections before drawing boxes.

### SQL Template

```sql
CREATE TABLE users (
  id        BIGINT PRIMARY KEY,
  username  VARCHAR(50) UNIQUE NOT NULL,
  email     VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE tweets (
  id         BIGINT PRIMARY KEY,
  user_id    BIGINT REFERENCES users(id),
  content    TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes to mention:
CREATE INDEX idx_tweets_user_id ON tweets(user_id);
CREATE INDEX idx_tweets_created_at ON tweets(created_at DESC);
```

### DB Schema Decisions to State Out Loud

- Why SQL vs NoSQL for this table?
- What's the shard key if you need to shard?
- Which columns need indexes (and why)?
- Will this table be read-heavy or write-heavy?

---

## Phase 3: High-Level Design (15 min)

### Start With This Skeleton

```
[Client] → [CDN (static)] / [API Gateway (dynamic)]
                 │
           [Load Balancer]
                 │
        [App Services / APIs]
          │           │
     [Cache]      [Databases]
                      │
               [Object Storage]
```

Then evolve it based on the specific problem.

### Key Components to Know Cold

| Component              | When to add                        |
| ---------------------- | ---------------------------------- |
| CDN                    | Images, static files, global users |
| Load Balancer          | Multiple app instances             |
| API Gateway            | Microservices, auth, rate limiting |
| Cache (Redis)          | Read-heavy, repeated queries       |
| Read Replicas          | Read:write > 5:1                   |
| Message Queue          | Async operations, decoupling       |
| Object Storage (S3)    | Images, videos, files              |
| Search (Elasticsearch) | Full-text search                   |

---

## Phase 4: Deep Dive (15 min)

Pick the 2-3 hardest parts and go deep.

### Common Deep Dive Areas

**For feed/timeline:**

- Fan-out on read vs write vs hybrid
- How do you handle celebrities (hotspots)?

**For storage:**

- Why this DB? SQL vs NoSQL? Schema design?
- How do you shard? What's the shard key?

**For reliability:**

- What happens when a service goes down?
- How do you handle duplicate messages?
- What's your retry strategy?

**For scale:**

- How does this handle 10× the traffic?
- Where's the bottleneck? How do you fix it?

---

## Phase 5: Trade-offs (5 min)

For every major decision, state:

```
"I chose X over Y because [reason].
The trade-off is [downside of X].
This is acceptable because [justification]."
```

### Common Trade-off Pairs

| Option A         | vs  | Option B        | Key Factor                    |
| ---------------- | --- | --------------- | ----------------------------- |
| SQL              | vs  | NoSQL           | Structure vs scale            |
| Consistency      | vs  | Availability    | CAP theorem                   |
| Fan-out on write | vs  | Fan-out on read | Read latency vs write fan-out |
| Sync replication | vs  | Async           | Durability vs latency         |
| Monolith         | vs  | Microservices   | Simplicity vs scale           |
| Cache-aside      | vs  | Write-through   | Freshness vs complexity       |

---

## Interviewer Signal Words

Watch for these hints:

| Interviewer Says                                | They Want You To                      |
| ----------------------------------------------- | ------------------------------------- |
| "How would this scale?"                         | Add caching, sharding, load balancing |
| "What if a server fails?"                       | Discuss HA, failover, replication     |
| "What if this is slow?"                         | Add caching, CDN, async processing    |
| "How would you ensure no data loss?"            | Replication, WAL, backups             |
| "What's the bottleneck?"                        | Identify weakest link, suggest fix    |
| "What would you do differently with more time?" | Discuss optimizations you skipped     |

---

## Common Mistakes

| Mistake                                 | Fix                                              |
| --------------------------------------- | ------------------------------------------------ |
| Jump to solution without clarifying     | Spend 5 min on requirements                      |
| Design for "infinite scale" immediately | Match design to stated scale                     |
| Ignore non-functional requirements      | Always discuss latency, availability             |
| Never state trade-offs                  | Every choice has a downside — say it             |
| Single server database                  | At any reasonable scale, think about replication |
| Ignore failure cases                    | "What if X fails?" is always fair game           |
| Use only one database                   | Most real systems are polyglot                   |

---

## Cheat Sheet: Scale → Architecture

```
< 1K users:       Single server + DB
< 10K users:      Separate app and DB servers
< 100K users:     Add caching (Redis), basic CDN
< 1M users:       Load balancer, read replicas, message queue
< 10M users:      DB sharding, microservices, multi-region CDN
< 100M users:     Everything distributed, multiple data centers
> 100M users:     You need a whole team of architects
```

---

## System Design Question Bank

### Beginner

- URL Shortener (bit.ly)
- TinyURL with analytics
- Pastebin

### Intermediate

- Design Twitter (timeline is the hard part)
- Design Instagram (photo sharing)
- Design Uber (real-time location)
- Design WhatsApp (message delivery)
- Notification System

### Advanced

- Design YouTube (video transcoding)
- Design Google Search
- Design a distributed cache (Redis)
- Design a rate limiter
- Design a distributed job scheduler

---

## Advanced Patterns (Deep Dive Reference)

### Resilience Patterns

- **Circuit Breaker** — stop calling a failing service; fail fast, recover gracefully → [circuit-breaker.md](../12-modern-patterns/circuit-breaker.md)
- **Retry with Backoff** — exponential backoff + jitter to avoid thundering herd → [failure-resilience.md](../16-failure-resilience/failure-resilience.md)
- **Dead Letter Queue (DLQ)** — capture messages that repeatedly fail processing
- **Graceful Degradation** — return cached/partial data instead of erroring out

### Coordination Patterns

- **Leader Election** — one node coordinates; use Zookeeper or Raft → [leader-election.md](../patterns/leader-election.md)
- **Distributed Locks** — Redis SETNX / Redlock for mutual exclusion → [distributed-locks.md](../12-modern-patterns/distributed-locks.md)
- **Distributed Cron** — leader-elected scheduler + idempotent jobs → [distributed-cron.md](../patterns/distributed-cron.md)
- **Idempotency Keys** — safe retries on writes; store result keyed by client ID → [idempotency.md](../12-modern-patterns/idempotency.md)

### Data Patterns

- **Cache Invalidation** — TTL vs event-driven vs write-through strategies → [cache-invalidation.md](../patterns/cache-invalidation.md)
- **Rate Limiter Designs** — token bucket, leaky bucket, sliding window → [rate-limiter-designs.md](../patterns/rate-limiter-designs.md)
- **Saga Pattern** — distributed transactions via choreography or orchestration → [saga-pattern.md](../07-microservices/saga-pattern.md)

### Observability

- **Metrics** — counters, gauges, histograms (Prometheus/CloudWatch)
- **Tracing** — distributed trace IDs across services (Jaeger, X-Ray)
- **Logging** — structured logs, correlation IDs, log aggregation → [observability.md](../12-modern-patterns/observability.md)

---

## Multi-Region & Global Scale

When the interviewer asks about global users or disaster recovery:

```
Strategy        | RTO  | RPO  | Cost
Active-Passive  | mins | secs | Medium
Active-Active   | secs | ~0   | High
Read Replicas   | n/a  | secs | Low (read-only)
```

- **RTO** (Recovery Time Objective) — how fast you recover
- **RPO** (Recovery Point Objective) — how much data you can afford to lose
- Geo-DNS routes users to nearest region
- Conflict resolution needed for active-active writes → [multi-region.md](../13-multi-region/multi-region.md)

---

## Edge Computing Consideration

Mention edge when: low latency is critical, content is personalized but cacheable, or you need to run logic close to the user.

- Cloudflare Workers, Lambda@Edge, Fastly Compute
- Good for: auth checks, A/B testing, geo-routing, image resizing
- Not good for: heavy compute, stateful operations, DB writes → [edge-computing.md](../15-edge-computing/edge-computing.md)

---

## Cost Optimization Notes (for "production-ready" signal)

Mentioning cost shows senior thinking:

- **Storage tiering** — hot (SSD) → warm (HDD) → cold (S3 Glacier)
- **Spot/preemptible instances** — for async workers and batch jobs
- **Caching as cost reduction** — fewer DB reads = lower bill
- **Right-sizing** — don't over-provision; use auto-scaling → [cost-optimization.md](../14-cost-optimization/cost-optimization.md)

---

## API Design — Advanced Signals

Things that impress interviewers at senior level:

- **Cursor-based pagination** over offset (consistent under inserts)
- **Partial responses** (`?fields=id,name`) to reduce bandwidth
- **Idempotency headers** (`Idempotency-Key`) on POST/PUT
- **GraphQL trade-offs** — flexible queries, but N+1 and caching complexity
- **Deprecation strategy** — sunset headers, versioned endpoints → [api-design-advanced.md](../17-api-design-advanced/api-design-advanced.md)

---

## Security Checklist (mention when relevant)

- Auth: JWT for stateless, OAuth for delegated, session tokens for stateful
- HTTPS everywhere; TLS termination at load balancer
- Rate limiting at API Gateway layer (per IP, per user)
- DDoS: CDN absorbs volumetric attacks; WAF for application-layer
- Input validation at the edge; never trust client data → [auth-jwt-oauth.md](../11-security/auth-jwt-oauth.md) | [ddos-protection.md](../11-security/ddos-protection.md)

---

## All Topic Notes

### Fundamentals

- [Client-Server Model](../01-fundamentals/client-server-model.md)
- [DNS](../01-fundamentals/dns.md)
- [HTTP & HTTPS](../01-fundamentals/http-https.md)
- [Back of Envelope](../01-fundamentals/back-of-envelope.md)

### Scaling

- [Vertical Scaling](../02-scaling/vertical-scaling.md)
- [Consistent Hashing](../02-scaling/consistent-hashing.md)

### Databases

- [SQL vs NoSQL](../03-databases/sql-vs-nosql.md)
- [Read Replicas](../03-databases/read-replicas.md)
- [Sharding](../03-databases/sharding.md)

### Networking

- [Load Balancers](../04-networking-and-routing/load-balancers.md)
- [API Gateway](../04-networking-and-routing/api-gateway.md)
- [CDN](../04-networking-and-routing/cdn.md)

### Caching & Async

- [Caching Strategies](../05-caching/caching-strategies.md)
- [Redis](../05-caching/redis.md)
- [Message Queues](../06-async-communication/message-queues-sqs.md)
- [Pub-Sub / Fan-out](../06-async-communication/pub-sub-sns.md)

### Microservices

- [Microservices](../07-microservices/microservices.md)
- [Containers & Docker](../07-microservices/containers-docker.md)
- [gRPC vs REST](../07-microservices/grpc-vs-rest.md)
- [Saga Pattern](../07-microservices/saga-pattern.md)
- [API Versioning](../07-microservices/api-versioning.md)

### Reliability & Performance

- [Rate Limiting](../08-reliability-and-performance/rate-limiting.md)
- [CAP Theorem](../08-reliability-and-performance/cap-theorem.md)
- [Bloom Filters](../08-reliability-and-performance/bloom-filters.md)

### HLD Case Studies

- [HLD: URL Shortener](../09-hld-case-studies/url-shortener.md)
- [HLD: Twitter](../09-hld-case-studies/twitter.md)
- [HLD: Notification System](../09-hld-case-studies/notification-system.md)
- [HLD: Instagram Feed](../09-hld-case-studies/instagram-feed.md)

### Security

- [Auth — JWT & OAuth](../11-security/auth-jwt-oauth.md)
- [DDoS Protection](../11-security/ddos-protection.md)

### Modern Patterns

- [Distributed Locks](../12-modern-patterns/distributed-locks.md)
- [Idempotency](../12-modern-patterns/idempotency.md)
- [Service Discovery](../12-modern-patterns/service-discovery.md)
- [Circuit Breaker](../12-modern-patterns/circuit-breaker.md)
- [Observability](../12-modern-patterns/observability.md)

### Advanced Topics

- [Multi-Region & Geo-Distribution](../13-multi-region/multi-region.md)
- [Cost Optimization](../14-cost-optimization/cost-optimization.md)
- [Edge Computing](../15-edge-computing/edge-computing.md)
- [Failure Handling & Resilience](../16-failure-resilience/failure-resilience.md)
- [API Design — Advanced](../17-api-design-advanced/api-design-advanced.md)

### Patterns

- [Leader Election](../patterns/leader-election.md)
- [Distributed Cron Jobs](../patterns/distributed-cron.md)
- [Rate Limiter Designs](../patterns/rate-limiter-designs.md)
- [Cache Invalidation](../patterns/cache-invalidation.md)

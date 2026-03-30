# 📊 Back of Envelope Calculations

> **One-liner:** Rough estimates done quickly to understand the scale of a system before designing it. They tell you what tier of infrastructure you need.

---

## 📌 Why This Matters

Before designing ANY system, you need to know:

- How many **users** will use it?
- How many **requests per second**?
- How much **storage** do we need?
- How much **bandwidth** is required?

These estimates shape every architectural decision.

---

## 🔢 Numbers Every Engineer Should Know

### Time

| Unit               | Value             |
| ------------------ | ----------------- |
| 1 millisecond      | 10⁻³ seconds      |
| 1 microsecond      | 10⁻⁶ seconds      |
| 1 nanosecond       | 10⁻⁹ seconds      |
| Seconds in a day   | ~86,400 ≈ 10⁵     |
| Seconds in a month | ~2.5M ≈ 2.5 × 10⁶ |
| Seconds in a year  | ~31.5M ≈ 3 × 10⁷  |

### Latency Numbers (Approximate)

| Operation                     | Latency    |
| ----------------------------- | ---------- |
| L1 cache reference            | 1 ns       |
| L2 cache reference            | 4 ns       |
| RAM reference                 | 100 ns     |
| SSD random read               | 150 µs     |
| HDD seek                      | 10 ms      |
| Network: same datacenter      | 500 µs     |
| Network: cross-region (US→EU) | 150 ms     |
| Network: cross-continent      | 200–300 ms |

### Storage Units

| Unit | Size       |
| ---- | ---------- |
| 1 KB | 10³ bytes  |
| 1 MB | 10⁶ bytes  |
| 1 GB | 10⁹ bytes  |
| 1 TB | 10¹² bytes |
| 1 PB | 10¹⁵ bytes |

---

## 📐 The Estimation Framework

### Step 1: DAU → QPS

```
Daily Active Users (DAU)    = 10 million
Avg requests per user/day   = 10
Total daily requests        = 100 million

QPS = 100M / 86,400 ≈ 1,200 req/sec
Peak QPS (2-3x average)     ≈ 3,000 req/sec
```

### Step 2: Storage

```
New posts per day    = 1M
Avg post size        = 1 KB (text) + 500 KB (image)
Daily storage        = 1M × 501 KB ≈ 500 GB/day
Yearly storage       = 500 GB × 365 ≈ 180 TB/year
5-year storage       ≈ 1 PB
```

### Step 3: Bandwidth

```
Reads per second     = 100K req/sec
Avg response size    = 10 KB
Outbound bandwidth   = 100K × 10 KB = 1 GB/sec
```

---

## 🔬 Real Example: Design Twitter

### Assumptions

- 300M DAU
- Each user reads 10 tweets/day
- Each user writes 1 tweet/week ≈ 0.14 tweets/day
- Avg tweet: 280 chars = 280 bytes ≈ 300 bytes
- 10% of tweets have an image (~200 KB)

### Read QPS

```
300M users × 10 reads/day = 3B reads/day
QPS = 3B / 86,400 ≈ 35,000 reads/sec
Peak QPS ≈ 100,000 reads/sec
```

### Write QPS

```
300M × 0.14 = 42M tweets/day
QPS = 42M / 86,400 ≈ 500 writes/sec
```

### Storage per day

```
Text: 42M × 300 bytes = 12.6 GB
Images: 42M × 10% × 200 KB = 840 GB
Total: ~850 GB/day ≈ 310 TB/year
```

### Bandwidth

```
Read traffic: 35K req/sec × 1 KB/response ≈ 35 MB/sec = ~300 Gbps
```

---

## 🔬 Real Example: Design WhatsApp

### Assumptions

- 2B DAU
- Each user sends 20 messages/day
- Avg message: 100 bytes
- 30% are media messages (500 KB avg)

### QPS

```
2B × 20 = 40B messages/day
QPS = 40B / 86,400 ≈ 460,000 msg/sec
Peak QPS ≈ 1M msg/sec
```

### Storage per day

```
Text: 40B × 100B = 4 TB
Media: 40B × 30% × 500 KB = 6 PB/day (too high → add retention/compression policy)
```

---

## 💡 Tips for Interviews

1. **Always ask** before estimating — "Should I assume 10M or 100M users?"
2. **Round aggressively** — 86,400 → 10⁵, 3.14 → 3
3. **Peak = 2-3× average** (or 5-10× for viral/event-based systems)
4. **Show your math** — interviewers care about the process, not the exact number
5. **Derive storage needs from estimates** — don't pull numbers from thin air
6. **1 char = 1 byte (ASCII)**, 1 char = 2-4 bytes (Unicode)

---

## 📋 Estimation Cheat Sheet

```
1K users  → Single server fine
10K users → Need to think about DB separation
100K users → Load balancer, read replicas
1M users  → Caching layer, CDN, sharding
10M users → Distributed systems, microservices
100M+     → You work at a FAANG (or interview there)
```

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/back-of-envelope.excalidraw`](../diagrams/back-of-envelope.excalidraw)

---

## 🔑 Key Takeaways

- Back-of-envelope is about **order of magnitude**, not precision
- Know the key numbers cold: bytes, seconds in a day, latency tiers
- Always separate **read QPS** from **write QPS** — they're usually very different
- Storage estimations reveal whether you need sharding/archival early

---

## 🔗 Related Topics

- [Horizontal Scaling](../02-scaling/horizontal-scaling.md)
- [Database Sharding](../03-databases/sharding.md)
- [CDN](../04-networking-and-routing/cdn.md)
- [Interview Framework](../10-interview-prep/framework.md)

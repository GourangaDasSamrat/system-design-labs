# 🌊 Fan-out Architecture

> **One-liner:** Fan-out means one event triggers multiple parallel downstream processes — like dropping a pebble in water and watching rings spread outward.

---

## 📌 The Pattern

```
[One Event] ──► [Fan-out Layer] ──► [Process A]
                                 ──► [Process B]
                                 ──► [Process C]
```

All processes run **simultaneously and independently**.

---

## 🔄 Fan-out on Write vs Fan-out on Read

This distinction is critical for feed/timeline systems (Twitter, Instagram).

### Fan-out on Write (Push)

When a user posts, immediately write to all followers' timelines:

```
User posts → find 1,000 followers → push to each follower's feed cache
Read: O(1) — just read pre-computed feed
Write: O(followers) — expensive for celebrities
```

### Fan-out on Read (Pull)

When a user opens their feed, compute it on the fly:

```
User opens feed → fetch followed users → merge their posts → sort
Read: O(following × posts) — slow at scale
Write: O(1) — just save the post
```

### Hybrid (Best for large systems)

- Normal users → fan-out on write
- Celebrities (>1M followers) → fan-out on read
- Merge both at read time

---

## 🌍 SNS → SQS Fan-out (AWS)

See [Pub-Sub (SNS)](./pub-sub-sns.md) for the full implementation.

The canonical pattern:

```
[Event] → [SNS Topic] → [SQS Queue A] → [Consumer A]
                      → [SQS Queue B] → [Consumer B]
                      → [SQS Queue C] → [Consumer C]
```

SQS between SNS and consumers adds durability and independent retry per consumer.

---

## 🔗 Related Topics

- [Pub-Sub (SNS)](./pub-sub-sns.md)
- [Message Queues (SQS)](./message-queues-sqs.md)
- [HLD: Twitter](../09-hld-case-studies/twitter.md)

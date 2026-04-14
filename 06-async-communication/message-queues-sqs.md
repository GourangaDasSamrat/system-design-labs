# 📬 Message Queues (SQS)

> **One-liner:** A message queue decouples services by letting a producer drop messages into a queue, where consumers pick them up asynchronously — neither needs to be available at the same time.

---

## 📌 The Problem: Tight Coupling

```
User places order →
  [Order Service] → calls [Payment Service] → waits →
                  → calls [Email Service] → waits →
                  → calls [Inventory Service] → waits →
  All done! (3s total, 3 failure points)
```

Problems:

- **High latency** — user waits for every downstream service
- **Cascading failure** — if Email service is down, order fails
- **Tight coupling** — Order service knows about all downstream services
- **No retry logic** — failures are permanent

---

## 💡 The Solution: Message Queue

```
User places order → [Order Service] → DROP message in queue → RESPOND (fast!)
                                              ↓
                                   [Queue]
                                   /    |    \
                          [Payment] [Email] [Inventory]  ← process at their own pace
```

Benefits:

- **Decoupling** — services don't know about each other
- **Resilience** — downstream service can be down; message waits in queue
- **Load leveling** — queue absorbs traffic spikes
- **Retry** — failed processing retried automatically

---

## 🔄 Queue Concepts

### Producer & Consumer

```
[Producer] → push message → [Queue] → pull message → [Consumer]
```

- **Producer**: Creates and sends messages
- **Consumer**: Reads and processes messages
- **Queue**: Durable buffer between them

### Message Lifecycle

```
1. Producer sends message → Queue stores it
2. Consumer polls queue → Queue marks message as "in-flight" (invisible to others)
3. Consumer processes → Success: Consumer deletes message
                     → Failure: After visibility timeout, message reappears
```

### Visibility Timeout

```
Consumer pulls message → message hidden for 30 seconds
If consumer crashes → 30 seconds later, message reappears → another consumer picks it up
If consumer succeeds → consumer deletes message → gone forever
```

---

## ☁️ AWS SQS (Simple Queue Service)

### Standard Queue

- **Unlimited throughput**
- **At-least-once delivery** — message may be delivered more than once!
- **Best-effort ordering** — no strict FIFO

### FIFO Queue

- **Exactly-once processing** — deduplication
- **Strict ordering** — first in, first out
- **Limited throughput** — 300 msg/sec (or 3000 with batching)

### SQS Key Concepts

```
Message retention:  Up to 14 days (default: 4 days)
Visibility timeout: How long consumer has to process (default: 30s)
Dead Letter Queue:  Where failed messages go after N retries
Max message size:   256 KB
```

### SQS + Lambda (Serverless Consumer)

```
[SQS Queue] ──triggers──► [Lambda Function]
                           (auto-scales to process backlog)
```

---

## 🔀 Queue Patterns

### Work Queue (Task Distribution)

```
[Producer] → [Queue] → [Worker 1]
                     → [Worker 2]
                     → [Worker 3]

Each message processed by exactly ONE worker
```

Use case: Image resizing, email sending, PDF generation

### Fan-out via Multiple Queues

```
[Event] → [SNS Topic] → [Queue A] → [Consumer A]
                      → [Queue B] → [Consumer B]
                      → [Queue C] → [Consumer C]
```

Each consumer gets a copy of the message. (See: SNS/Pub-Sub)

### Priority Queue

```
High priority queue → processed first by consumers
Low priority queue  → processed when high is empty
```

---

## ♻️ Dead Letter Queue (DLQ)

When a message fails to process after N retries → move to DLQ:

```
[Queue] → [Consumer fails 3 times] → [DLQ]
                                          ↑
                                    Ops team inspects,
                                    fixes bug, redrives
```

```json
// SQS Redrive Policy
{
  "deadLetterTargetArn": "arn:aws:sqs:us-east-1:123:my-dlq",
  "maxReceiveCount": 3 // after 3 failures, move to DLQ
}
```

---

## ⚠️ Idempotency — Critical Requirement

Standard queues deliver **at-least-once**. Your consumer MUST be idempotent:

```
Message: "charge user 42 for $99"

❌ Non-idempotent: Just run the charge every time → user charged twice!
✅ Idempotent: Check if orderId already processed → skip if yes
```

```javascript
async function processPayment(message) {
  const { orderId, userId, amount } = message;

  // Check idempotency key
  const alreadyProcessed = await redis.get(`payment:${orderId}`);
  if (alreadyProcessed) return; // skip duplicate

  await chargeUser(userId, amount);
  await redis.setex(`payment:${orderId}`, 86400, "1"); // mark done
}
```

---

## 📊 SQS vs Other Queues

| Feature    | AWS SQS          | RabbitMQ           | Apache Kafka            |
| ---------- | ---------------- | ------------------ | ----------------------- |
| Type       | Managed cloud    | Self-hosted broker | Distributed log         |
| Retention  | 14 days          | Until consumed     | Configurable (forever)  |
| Ordering   | Best-effort/FIFO | Per-queue          | Per-partition           |
| Replay     | ❌               | ❌                 | ✅ (replay from offset) |
| Throughput | Very high        | High               | Very high               |
| Use case   | Decoupling tasks | Complex routing    | Event streaming         |

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/message-queues.excalidraw`](../diagrams/message-queues.excalidraw)

The diagram shows:

- Producer → Queue → Multiple consumers
- Visibility timeout flow (in-flight → reappear)
- Dead Letter Queue after max retries
- Fan-out via SNS → multiple queues

---

## 🔑 Key Takeaways

- Queues **decouple** services and absorb traffic spikes
- **Visibility timeout** ensures messages aren't lost if a consumer crashes
- Always implement **idempotent consumers** with SQS Standard
- **DLQ** is essential for debugging failed messages
- Use **FIFO** when order matters; Standard when throughput matters

---

## 🔗 Related Topics

- [Pub-Sub (SNS)](./pub-sub-sns.md)
- [Fan-out Architecture](./fan-out.md)
- [Microservices](../07-microservices/microservices.md)
- [Rate Limiting](../08-reliability-and-performance/rate-limiting.md)

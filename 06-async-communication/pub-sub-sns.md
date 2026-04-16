# 📢 Pub-Sub (SNS) & Fan-out Architecture

> **One-liner:** Pub-Sub lets publishers broadcast events to all interested subscribers without knowing who they are. Fan-out ensures one event triggers many parallel actions.

---

## 📌 Pub-Sub Model

In traditional queues: one message → one consumer.

In Pub-Sub: one message → **all subscribers** receive a copy.

```
[Publisher] → [Topic] → [Subscriber A]
                      → [Subscriber B]
                      → [Subscriber C]
```

---

## ☁️ AWS SNS (Simple Notification Service)

SNS is AWS's managed Pub-Sub service.

### Key Concepts

| Term             | Meaning                                        |
| ---------------- | ---------------------------------------------- |
| **Topic**        | A named channel that messages are published to |
| **Publisher**    | Sends messages to a topic                      |
| **Subscriber**   | Receives copies of messages                    |
| **Subscription** | A subscriber's registration to a topic         |

### Supported Subscriber Types

- **SQS Queue** — most common (durable, retry-able)
- **Lambda** — serverless processing
- **HTTP/HTTPS endpoint** — webhooks
- **Email / SMS** — direct notifications
- **Mobile Push** (FCM, APNs) — push notifications

---

## 🔄 SNS Message Flow

```
Order Placed Event
      ↓
[SNS Topic: order-events]
      ├──► [SQS Queue] → [Payment Lambda]
      ├──► [SQS Queue] → [Email Service]
      ├──► [SQS Queue] → [Inventory Service]
      └──► [Lambda]    → [Analytics Ingestion]
```

All four happen **simultaneously and independently**.

---

## 🌟 Fan-out Architecture

Fan-out = one event → many parallel processes

### SNS → SQS Fan-out Pattern (Most Robust)

```
[Event Source]
      │
      ▼
[SNS Topic]
   │    │    │
   ▼    ▼    ▼
[SQS] [SQS] [SQS]
  │     │     │
  ▼     ▼     ▼
[Svc A][Svc B][Svc C]
```

Why SQS in between SNS?

- **Durability** — if Service A is down, message waits in SQS
- **Rate control** — each service processes at its own pace
- **Retry** — SQS handles retries independently per service
- **Decoupling** — SNS doesn't care about downstream state

### Direct SNS Fan-out (No SQS buffer)

```
[SNS Topic] → [Lambda A]  (immediate, no buffer)
            → [Lambda B]
```

Risk: If Lambda B fails, message is lost (no retry queue).

---

## 🏗️ Real-World Example: User Registration

```
User signs up → [Auth Service] → publishes to SNS: "user.created"
                                          │
                              ┌──────────┼──────────┐
                              ▼          ▼          ▼
                         [SQS]→     [SQS]→     [SQS]→
                         [Welcome   [Analytics  [Referral
                          Email]     Track]      Credit]
```

Each service is independent. If email service is down → message waits → sent later.
Adding a 4th service (e.g., Slack alert)? Just add a new SNS subscription. Zero code change in Auth Service.

---

## 🆚 SNS vs SQS vs Kafka

| Feature           | SNS                    | SQS                     | Kafka                        |
| ----------------- | ---------------------- | ----------------------- | ---------------------------- |
| Pattern           | Pub-Sub (broadcast)    | Queue (single consumer) | Log (many consumers, replay) |
| Message retention | None (fire and forget) | Up to 14 days           | Configurable (forever)       |
| Fan-out           | ✅ Native              | ❌ (one consumer)       | ✅ (consumer groups)         |
| Replay            | ❌                     | ❌                      | ✅                           |
| Ordering          | Best effort            | FIFO option             | Per-partition                |
| Durability        | With SQS subscription  | ✅                      | ✅                           |
| Managed           | ✅ AWS                 | ✅ AWS                  | Self-hosted / Confluent      |

---

## 🔔 SNS for Push Notifications

SNS handles mobile push at scale:

```
[Your Server] → SNS publishes to platform endpoint
                     ↓
             [FCM] (Android) → User's phone
             [APNs] (iOS) → User's phone
```

SNS manages platform credentials, token management, delivery.

---

## 📊 Message Filtering (SNS)

Subscribers can filter messages by attributes:

```json
// SNS message with attributes
{
  "order_type": "premium",
  "region": "us-east",
  "amount": 999
}

// Subscription filter policy (Premium SQS only gets premium orders)
{
  "order_type": ["premium"]
}
```

Only premium orders go to the premium processing queue — others are filtered out.

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/pub-sub-fan-out.excalidraw`](../diagrams/pub-sub-fan-out.excalidraw)

The diagram shows:

- Publisher → SNS Topic → multiple SQS queues → multiple consumers
- Fan-out flow on user.created event
- Message filtering at subscription level
- DLQ for each consumer independently

---

## 🔑 Key Takeaways

- **Pub-Sub** (SNS) = broadcast; **Queue** (SQS) = point-to-point
- Always use **SNS → SQS fan-out** for durability (SQS buffers failures)
- Pub-Sub enables **zero-code service addition** — new subscribers need no changes to publisher
- **Message filtering** reduces unnecessary processing across services

---

## 🔗 Related Topics

- [Message Queues (SQS)](./message-queues-sqs.md)
- [Microservices](../07-microservices/microservices.md)
- [Notification System HLD](../09-hld-case-studies/notification-system.md)

# 🔔 HLD: Notification System

> Design a notification system that sends push notifications, emails, and SMS to users based on events in the platform.

---

## 1️⃣ Clarify Requirements

### Functional Requirements

- Send notifications via: Push (iOS/Android), Email, SMS
- Trigger types: real-time (someone liked your post) and scheduled (weekly digest)
- Users can set preferences: opt-in/out per channel and notification type
- Support for **templated** notifications (not hardcoded)
- Delivery guarantee — important notifications must not be dropped

### Non-Functional Requirements

- **High throughput** — 10M notifications/day
- **Low latency** for real-time notifications (< 5 seconds)
- **Reliability** — retry on failure
- **Scalability** — handle spikes (breaking news, product launches)
- **Observability** — track delivered/failed/opened

---

## 2️⃣ Estimate Scale

```
Notifications/day: 10M
Notifications/sec: 10M / 86,400 ≈ 116/sec (avg)
Peak: 1,000/sec (event spikes)

Breakdown:
  Push: 60% → 6M/day
  Email: 30% → 3M/day
  SMS: 10% → 1M/day

Storage (notification log):
  1 record ≈ 500 bytes
  10M/day × 500B = 5 GB/day
  Retention: 90 days → 450 GB
```

---

## 3️⃣ API Design

```
# Send a notification (internal service call)
POST /notifications/send
Body: {
  "userId": "user_123",
  "type": "like_received",
  "channels": ["push", "email"],  // or leave empty → use user preferences
  "data": {
    "actorName": "Rahul",
    "postTitle": "My Design Notes"
  }
}

# User preference management
GET  /users/:userId/notification-preferences
PUT  /users/:userId/notification-preferences
Body: {
  "push": { "like_received": true, "new_follower": true, "marketing": false },
  "email": { "weekly_digest": true, "marketing": false }
}

# Scheduled notifications
POST /notifications/schedule
Body: {
  "templateId": "weekly_digest",
  "scheduledAt": "2024-01-15T09:00:00Z",
  "userSegment": "active_users"
}
```

---

## 4️⃣ High-Level Architecture

```
[Event Sources]
  - User Service ("user A liked post")
  - Order Service ("your order shipped")
  - Scheduler ("send weekly digest")
        │
        ▼
[Notification Service API]
        │
        ├─► Check User Preferences → skip if opted out
        ├─► Build notification from template
        ├─► Route to appropriate channels
        │
        ▼
[Message Queue (Kafka / SQS)]
  ├──► [Push Queue]
  ├──► [Email Queue]
  └──► [SMS Queue]
        │
        ▼
[Channel Workers]
  ├──► [Push Worker] → FCM (Android) / APNs (iOS)
  ├──► [Email Worker] → SendGrid / SES / Mailgun
  └──► [SMS Worker] → Twilio / SNS
        │
        ▼
[Delivery Log DB] ← track status per notification
```

---

## 5️⃣ Component Deep Dives

### Preference Service

```python
# User preferences (stored in DB, cached in Redis)
preferences = {
  "push": {
    "like_received": True,
    "new_follower": True,
    "marketing": False
  },
  "email": {
    "weekly_digest": True,
    "security_alert": True,
    "marketing": False
  },
  "sms": {
    "security_alert": True,
    "marketing": False
  }
}

def should_notify(userId, notifType, channel):
    prefs = redis.get(f"prefs:{userId}") or db.get(userId)
    return prefs.get(channel, {}).get(notifType, True)  # default: on
```

### Template Service

```json
// Template: "like_received"
{
  "templateId": "like_received",
  "push": {
    "title": "{{actorName}} liked your post",
    "body": "\"{{postTitle}}\" is getting attention!"
  },
  "email": {
    "subject": "{{actorName}} liked your post",
    "templateFile": "like-received.html"
  }
}
```

Render: inject actual values, produce final notification content.

### Push Notification Worker

```python
def send_push(notification):
    device_tokens = db.get_device_tokens(notification.userId)

    for token in device_tokens:
        if token.platform == "android":
            fcm.send({
                "token": token.value,
                "title": notification.title,
                "body": notification.body,
                "data": notification.data
            })
        elif token.platform == "ios":
            apns.send({
                "deviceToken": token.value,
                "aps": { "alert": { "title": notification.title, "body": notification.body } }
            })
```

### Retry & Dead Letter

```
Worker fails to send → retry with exponential backoff
  Attempt 1: immediate
  Attempt 2: 30 seconds
  Attempt 3: 5 minutes
  Attempt 4: 30 minutes
  Attempt 5: DLQ (manual inspection or discard)
```

---

## 6️⃣ Database Schema

```sql
-- Notification log (Cassandra — write-heavy, time-series)
CREATE TABLE notifications (
  notification_id  UUID,
  user_id          BIGINT,
  type             VARCHAR(50),
  channel          VARCHAR(20),  -- push/email/sms
  status           VARCHAR(20),  -- pending/sent/failed/delivered/opened
  payload          JSONB,
  created_at       TIMESTAMP,
  sent_at          TIMESTAMP,
  PRIMARY KEY (user_id, created_at, notification_id)
) WITH CLUSTERING ORDER BY (created_at DESC);

-- User device tokens
CREATE TABLE device_tokens (
  user_id    BIGINT,
  token      TEXT,
  platform   VARCHAR(10),  -- ios/android/web
  created_at TIMESTAMP,
  PRIMARY KEY (user_id, token)
);

-- User preferences (PostgreSQL, read-heavy)
CREATE TABLE notification_preferences (
  user_id     BIGINT PRIMARY KEY,
  preferences JSONB,
  updated_at  TIMESTAMP
);
```

---

## 7️⃣ Handling Scale Spikes

### Event: Product Launch → 10M users notified

Problem: Sudden flood of 10M notifications in seconds.

Solution: **Rate limiting the outbound queue**

```
Email: SendGrid allows 1000 emails/sec
→ Queue backs up → workers drain at 1000/sec → all 10M sent in ~3 hours
→ Users get it "within a few hours" → acceptable for marketing
```

### Priority Queues

```
HIGH priority queue:   security alerts, OTPs → process immediately
MEDIUM priority queue: social notifications → within seconds
LOW priority queue:    marketing, digests → within hours
```

Workers check HIGH first, then MEDIUM, then LOW.

---

## 8️⃣ Observability

Track every notification:

```
Events to log:
  - notification_queued     (timestamp, userId, type, channel)
  - notification_sent       (timestamp, provider response)
  - notification_delivered  (timestamp, from FCM/APNs delivery receipt)
  - notification_opened     (timestamp, from SDK tracking)
  - notification_failed     (timestamp, error, retryCount)

Metrics:
  - Delivery rate per channel
  - Average send latency (queued → sent)
  - Failure rate per provider
  - Open rate per notification type
```

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/hld-notification-system.excalidraw`](../diagrams/hld-notification-system.excalidraw)

The diagram shows:

- Event sources → Notification Service → Kafka queues → channel workers → external providers
- Preference check before queuing
- Template rendering step
- Retry with DLQ
- Priority queue lanes (high/medium/low)
- Delivery status tracking

---

## ✅ Trade-offs Summary

| Decision           | Choice              | Rationale                                    |
| ------------------ | ------------------- | -------------------------------------------- |
| Queue type         | Kafka/SQS           | Decouple events from delivery, absorb spikes |
| Priority           | Multiple queues     | Don't let marketing delay OTPs               |
| Retry              | Exponential backoff | Avoid hammering failed providers             |
| Preference storage | Redis cache + DB    | Fast reads on every notification             |
| Delivery tracking  | Cassandra           | High-volume, time-series write pattern       |

---

## 🔗 Related Topics

- [Message Queues (SQS)](../06-async-communication/message-queues-sqs.md)
- [Pub-Sub (SNS)](../06-async-communication/pub-sub-sns.md)
- [Rate Limiting](../08-reliability-and-performance/rate-limiting.md)
- [Microservices](../07-microservices/microservices.md)

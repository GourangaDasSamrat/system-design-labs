# 🔁 Idempotency

> **One-liner:** An operation is idempotent if doing it multiple times produces the same result as doing it once — critical for safe retries in distributed systems.

> 🎨 **Diagram:** [idempotency.excalidraw](../diagrams/idempotency.excalidraw) — open in [Excalidraw](https://excalidraw.com) (File → Open)

---

## ❓ Why Does This Matter?

In distributed systems, **anything can fail at any time**:

- Network times out — did the server receive the request?
- Client retries — does the server process it again?
- Message queue delivers a message twice

**Without idempotency:** User clicks "Pay" → network blips → client retries → user charged twice. 💸

---

## 📐 Idempotent vs Non-Idempotent

| Operation                      | Idempotent? | Why                            |
| ------------------------------ | ----------- | ------------------------------ |
| `GET /users/1`                 | ✅ Yes      | Same response every time       |
| `PUT /users/1 {name: "Ali"}`   | ✅ Yes      | Setting to same value          |
| `DELETE /users/1`              | ✅ Yes      | Second delete = already gone   |
| `POST /payments {amount: 100}` | ❌ No       | Creates a new charge each time |
| `PATCH /account/balance -=100` | ❌ No       | Each call subtracts again      |

---

## 🔑 Idempotency Keys

The standard solution for non-idempotent operations:

```
Client generates:  idempotency-key: uuid-abc-123

POST /payments
Headers:
  Idempotency-Key: uuid-abc-123
Body:
  { amount: 100, currency: "USD" }
```

**Server logic:**

```
1. Receive request with idempotency-key
2. Check: have we seen this key before?
   NO  → Process payment, store result with key (TTL: 24h)
   YES → Return cached response (don't charge again)
3. Return result
```

**Storage example (Redis):**

```bash
# On first request:
SET idem:uuid-abc-123 '{"status":"ok","chargeId":"ch_123"}' EX 86400

# On retry:
GET idem:uuid-abc-123  # Returns cached response immediately
```

---

## 🏗️ Database-Level Idempotency

### Unique Constraints

```sql
-- Prevent duplicate order creation
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  idempotency_key VARCHAR(255) UNIQUE,  -- ← key here
  user_id INT,
  amount DECIMAL,
  created_at TIMESTAMP
);

INSERT INTO orders (id, idempotency_key, user_id, amount)
VALUES (gen_random_uuid(), 'uuid-abc-123', 42, 100.00)
ON CONFLICT (idempotency_key) DO NOTHING;
--                                         ↑ safe retry!
```

### Conditional Updates (Optimistic Locking)

```sql
UPDATE accounts
SET balance = balance - 100, version = version + 1
WHERE id = 42 AND version = 7;
-- If version changed, 0 rows updated → retry with fresh data
```

---

## 🔄 Idempotency in Message Queues

SQS, Kafka — delivery guarantees are **at-least-once**. Your consumers must be idempotent.

```javascript
async function processPaymentEvent(message) {
  const { paymentId, amount } = message;

  // Check if already processed
  const exists = await db.query(
    "SELECT 1 FROM processed_payments WHERE payment_id = $1",
    [paymentId],
  );
  if (exists.rows.length > 0) return; // Already done, skip

  // Process payment
  await chargeUser(amount);

  // Mark as processed
  await db.query("INSERT INTO processed_payments (payment_id) VALUES ($1)", [
    paymentId,
  ]);
}
```

---

## ✅ Pros

- Safe retries without side effects
- Enables reliable distributed workflows
- Standard Stripe/PayPal/Twilio all use idempotency keys
- Works naturally with at-least-once delivery

## ❌ Cons

- Extra storage needed for idempotency key cache
- Key management adds complexity (TTL choices, cleanup)
- Doesn't help if operations must be strictly ordered
- Race condition: two parallel requests with same key can both pass check simultaneously → need atomic check-and-set

## ⚖️ When to Use / When NOT to Use

**✅ Use when:**

- Payment processing, order creation (any financial operation)
- Email/SMS sending (prevent duplicates)
- Message queue consumers (at-least-once delivery)
- Any API that clients will retry on failure

**❌ Avoid / not needed when:**

- Pure reads (already idempotent by nature)
- Strictly ordered append-only logs
- Operations where duplicates are intentional (e.g., add to cart)

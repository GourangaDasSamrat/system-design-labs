# 🔄 Data Consistency in Microservices — Saga Pattern

> **One-liner:** In microservices, you can't use a single database transaction across services — the Saga pattern breaks it into a sequence of local transactions with compensating rollbacks on failure.

> 🎨 **Diagram:** [saga-pattern.excalidraw](../diagrams/saga-pattern.excalidraw) — open in [Excalidraw](https://excalidraw.com) (File → Open)

---

## ❓ The Problem: Distributed Transactions

In a monolith, you can wrap everything in one DB transaction:

```sql
BEGIN;
  UPDATE orders SET status='confirmed' WHERE id=1;
  UPDATE inventory SET stock=stock-1 WHERE product_id=5;
  INSERT INTO payments (order_id, amount) VALUES (1, 99.99);
COMMIT; -- all or nothing
```

In microservices, each service owns its own DB. **You cannot do a cross-service ACID transaction.** If the payment service fails after inventory was deducted, you're in an inconsistent state.

---

## 🎭 The Saga Pattern

A Saga is a sequence of local transactions. Each step publishes an event triggering the next. If any step fails, **compensating transactions** undo the previous steps.

### Example: E-commerce Order Flow

```
Step 1: Order Service      → Create order (PENDING)
Step 2: Inventory Service  → Reserve items
Step 3: Payment Service    → Charge customer
Step 4: Shipping Service   → Schedule delivery
Step 5: Order Service      → Mark order CONFIRMED
```

**Happy path ✅:**

```
Order Created → Items Reserved → Payment Charged → Shipping Scheduled → Done ✅
```

**Failure at Payment ❌:**

```
Order Created → Items Reserved → Payment FAILED
                                       ↓
                          Compensate: Release inventory
                          Compensate: Cancel order
```

---

## 🏗️ Two Implementation Styles

### 1. Choreography (Event-driven, decentralized)

Each service listens for events and reacts. No central coordinator.

```
OrderService    ──publishes──► "OrderCreated"
InventoryService ──listens──► reserves stock, publishes "StockReserved"
PaymentService  ──listens──► charges card, publishes "PaymentProcessed"
ShippingService ──listens──► schedules delivery, publishes "Shipped"

On failure:
PaymentService publishes "PaymentFailed"
InventoryService listens, releases stock, publishes "StockReleased"
OrderService listens, marks order CANCELLED
```

**Pros:** Simple, decoupled, no single point of failure  
**Cons:** Hard to track overall saga state, cyclic dependencies possible

---

### 2. Orchestration (Centralized coordinator)

A dedicated Saga Orchestrator tells each service what to do.

```
                    ┌─────────────────────┐
                    │   Saga Orchestrator  │
                    │   (Order Saga)       │
                    └──────────┬──────────┘
                               │
         ┌─────────────────────┼──────────────────────┐
         ▼                     ▼                       ▼
  InventoryService       PaymentService          ShippingService
  "Reserve stock"        "Charge $99"            "Ship to addr"
  → "Reserved" ✅        → "Failed" ❌           (not called)
         │                     │
         └─────────────────────┘
                    Orchestrator receives "Failed"
                    → tells InventoryService: "Release stock"
                    → marks saga FAILED
```

**Pros:** Clear saga state, easy to audit and debug  
**Cons:** Orchestrator is a bottleneck / SPOF if not designed carefully

---

## 🆚 Choreography vs Orchestration

|                | Choreography                  | Orchestration            |
| -------------- | ----------------------------- | ------------------------ |
| **Control**    | Decentralized                 | Centralized              |
| **Coupling**   | Loose (events)                | Tighter (direct calls)   |
| **Visibility** | Hard to trace end-to-end      | Easy — one place to look |
| **Complexity** | Low per-service, high overall | High orchestrator logic  |
| **Best for**   | 3-4 step sagas                | Complex, many-step flows |

---

## 💾 Saga State Storage

Track saga state to survive crashes:

```sql
CREATE TABLE sagas (
  id UUID PRIMARY KEY,
  type VARCHAR(50),          -- 'order_saga'
  current_step VARCHAR(50),  -- 'awaiting_payment'
  status VARCHAR(20),        -- 'IN_PROGRESS', 'COMPLETED', 'FAILED'
  payload JSONB,             -- order details
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);
```

---

## ✅ Pros

- Maintains data consistency without distributed transactions
- Works with polyglot persistence (each service uses its own DB)
- Choreography scales well horizontally
- Orchestration provides clear audit trail

## ❌ Cons

- Eventual consistency — data temporarily inconsistent between steps
- Compensating transactions are complex to implement correctly
- Hard to debug when something fails mid-saga
- Idempotency required for every step (retries happen)

## ⚖️ When to Use / When NOT to Use

**✅ Use when:**

- Multi-service workflows that must be atomic (order → payment → shipping)
- Different services own different databases
- Long-running business processes (insurance claims, loan approvals)

**❌ Avoid when:**

- Single service with one DB (just use a DB transaction)
- Simple async operations that don't need rollback
- You can redesign to avoid the distributed write entirely

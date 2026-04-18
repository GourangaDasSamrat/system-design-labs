# 🧩 Microservices

> **One-liner:** Instead of one giant application (monolith), microservices split the system into small, independent services — each owning its own data and deployed separately.

---

## 📌 Monolith vs Microservices

### Monolith

```
[One Big App]
 ├── User Module
 ├── Order Module
 ├── Payment Module
 ├── Notification Module
 └── [One Database]
```

Deploy anything → redeploy everything. One bug → whole app down.

### Microservices

```
[User Service]         → [Users DB]
[Order Service]        → [Orders DB]
[Payment Service]      → [Payments DB]
[Notification Service] → [Notif DB]
```

Each service: independent codebase, independent deployment, independent database.

---

## ✅ Pros of Microservices

| Benefit                    | Detail                                        |
| -------------------------- | --------------------------------------------- |
| **Independent deployment** | Deploy Payment without touching User Service  |
| **Independent scaling**    | Scale only the Order Service during sales     |
| **Tech diversity**         | Order Service in Go, ML Service in Python     |
| **Fault isolation**        | Notification Service down → Orders still work |
| **Small teams**            | Each team owns one service (Conway's Law)     |
| **Easier to understand**   | Each service is small and focused             |

## ❌ Cons of Microservices

| Problem                    | Detail                                               |
| -------------------------- | ---------------------------------------------------- |
| **Network overhead**       | In-process calls → network calls (latency + failure) |
| **Distributed tracing**    | Debugging a request across 10 services is hard       |
| **Data consistency**       | No cross-service ACID transactions                   |
| **Operational complexity** | 20 services = 20 deployments, 20 logs, 20 monitors   |
| **Service discovery**      | How does Service A find Service B?                   |
| **Testing**                | Integration testing is much harder                   |

---

## 🔄 Inter-Service Communication

### Synchronous (Request-Response)

```
Order Service → HTTP/gRPC → Payment Service → response
```

✅ Simple, immediate response  
❌ Tight temporal coupling — both must be up  
❌ Cascading failures

**When to use:** When the caller needs the result immediately (e.g., payment confirmation before showing success page)

### Asynchronous (Event-Driven)

```
Order Service → [Event Bus / Queue] → Payment Service
                                    → Email Service
```

✅ Decoupled — services can be down  
✅ Natural fan-out  
❌ Eventual consistency  
❌ Harder to debug

**When to use:** Notifications, analytics, side effects that don't block the user

---

## 🏗️ Service Communication Patterns

### API Gateway Pattern

```
Client → [API Gateway] → routes to appropriate service
                       → handles auth, rate limiting for all
```

### Service Mesh (Advanced)

```
[Service A] ←──[sidecar proxy]──[sidecar proxy]──► [Service B]
                  (Envoy)              (Envoy)
```

Sidecar handles: mTLS, retries, timeouts, circuit breaking, observability.
Examples: Istio, Linkerd, AWS App Mesh.

### BFF (Backend for Frontend)

```
[Mobile App] → [Mobile BFF] → aggregates calls to services
[Web App]    → [Web BFF]    → different data shape per client
```

---

## 🗄️ Database Per Service

Each service **owns its data**. No shared DB.

```
❌ Wrong:
[User Service]  ──┐
[Order Service] ──┼──► [Shared DB]  ← tight coupling at data level
[Payment Service] ┘

✅ Right:
[User Service]    → [User DB (PostgreSQL)]
[Order Service]   → [Order DB (MongoDB)]
[Payment Service] → [Payment DB (PostgreSQL)]
```

**But what about joins?**

- Denormalize data (copy what you need)
- Use API calls to get data from another service
- Use event-driven data replication (each service maintains its own read model)

---

## 🔒 Saga Pattern — Distributed Transactions

No cross-service ACID transactions. Use **Sagas** instead.

### Choreography Saga (Event-Driven)

```
Order Created →
  Payment Service listens → charges card → Payment Succeeded event
  Inventory Service listens → reserves stock → Stock Reserved event
  Email Service listens → sends confirmation

If payment fails → Payment Failed event →
  Order Service listens → cancels order
```

### Orchestration Saga (Central Coordinator)

```
[Saga Orchestrator]
  1. → tell Payment Service to charge
  2. → tell Inventory to reserve
  3. → tell Email to send confirmation

If step 2 fails:
  → tell Payment Service to refund (compensating transaction)
```

---

## 🌡️ Circuit Breaker Pattern

Prevent cascading failures:

```
Normal:   Service A → Service B (works fine)
Degraded: Service A → Service B (Service B is slow/erroring)
Open:     Service A → [Circuit Open] → return fallback (don't call B)
          Wait 30 seconds → try again (half-open)
          If OK → close circuit → normal again
```

States: **Closed** (normal) → **Open** (failing, reject calls) → **Half-Open** (testing recovery)

Libraries: Hystrix (Java), resilience4j, Polly (.NET), opossum (Node.js)

---

## 🔍 Observability in Microservices

### Distributed Tracing

```
Request ID: abc-123
→ API Gateway     [10ms]
→ Order Service   [50ms]
→ Payment Service [200ms]  ← slow here!
→ Email Service   [30ms]
Total: 290ms
```

Tools: Jaeger, Zipkin, AWS X-Ray, Datadog APM

### Centralized Logging

All services ship logs to one place: ELK Stack (Elasticsearch + Logstash + Kibana), Datadog, CloudWatch.

### Metrics & Alerting

Prometheus + Grafana: CPU, memory, request rate, error rate, latency (p50, p95, p99).

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/microservices.excalidraw`](../diagrams/microservices.excalidraw)

The diagram shows:

- Monolith vs microservices comparison
- API Gateway + service mesh layout
- Saga orchestration flow
- Circuit breaker states

---

## 🔑 Key Takeaways

- **Don't start with microservices** — start with a well-structured monolith and extract services when you hit scaling/team pain
- **Database per service** is non-negotiable for true independence
- **Sagas** replace distributed transactions (embrace eventual consistency)
- **Circuit breakers** are essential — one slow service will take down the whole system without them
- Observability (tracing, logging, metrics) is twice as important in microservices

---

## 🔗 Related Topics

- [API Gateway](../04-networking-and-routing/api-gateway.md)
- [Message Queues (SQS)](../06-async-communication/message-queues-sqs.md)
- [Pub-Sub (SNS)](../06-async-communication/pub-sub-sns.md)
- [Containers & Docker](./containers-docker.md)

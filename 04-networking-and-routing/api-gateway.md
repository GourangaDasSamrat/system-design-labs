# 🚪 API Gateway

> **One-liner:** A single entry point for all client requests that handles cross-cutting concerns like auth, rate limiting, routing, and logging — so your microservices don't have to.

---

## 📌 The Problem

In a microservices architecture, clients would need to call many services directly:

```
Mobile App → auth-service:3001
           → user-service:3002
           → order-service:3003
           → payment-service:3004
           → notification-service:3005
```

Problems:

- Client needs to know every service's address
- Auth logic duplicated in every service
- CORS, rate limiting repeated everywhere
- No single point for monitoring

---

## 💡 The Solution: API Gateway

```
Mobile App ──────────────► [API Gateway]
                                │
                                ├─► auth-service
                                ├─► user-service
                                ├─► order-service
                                └─► payment-service
```

The gateway is the **single front door** to your backend.

---

## 🛠️ What API Gateways Do

### 1. Request Routing

```
POST /auth/login      → auth-service
GET  /users/:id       → user-service
POST /orders          → order-service
GET  /products        → product-service
```

### 2. Authentication & Authorization

```
Client → [Gateway]
          → Validate JWT token
          → If invalid: 401 Unauthorized (request never hits services)
          → If valid: forward with user context
```

### 3. Rate Limiting

```
User A: 100 requests/min → allow
User A: 101st request    → 429 Too Many Requests
```

### 4. SSL Termination

Handles HTTPS at the gateway; internal traffic can be HTTP.

### 5. Request/Response Transformation

```
Client sends:  { "user_id": 123 }
Gateway transforms to: { "userId": "user_123", "timestamp": "..." }
```

### 6. Load Balancing

Routes to healthy instances of each microservice.

### 7. Caching

Cache responses at the gateway level for frequently accessed data.

### 8. Logging & Monitoring

Single place to log all incoming requests, response times, error rates.

### 9. Circuit Breaking

If a downstream service is failing, stop sending requests → return cached/fallback response.

### 10. API Versioning

```
/v1/users → old-user-service (v1)
/v2/users → new-user-service (v2)
```

---

## 🔄 Request Lifecycle Through Gateway

```
Client Request
    │
    ▼
[1] SSL Termination
    │
    ▼
[2] Authentication (JWT/OAuth validation)
    │
    ▼
[3] Rate Limiting Check
    │
    ▼
[4] Request Routing (which service?)
    │
    ▼
[5] Request Transformation (headers, body)
    │
    ▼
[6] Forward to Microservice
    │
    ▼
[7] Response Transformation
    │
    ▼
[8] Logging & Metrics
    │
    ▼
Client Response
```

---

## ⚖️ API Gateway vs Load Balancer

| Feature                | Load Balancer      | API Gateway                            |
| ---------------------- | ------------------ | -------------------------------------- |
| Primary role           | Distribute traffic | Smart routing + cross-cutting concerns |
| Layer                  | L4 or L7           | L7 (always)                            |
| Authentication         | ❌                 | ✅                                     |
| Rate limiting          | Limited            | ✅                                     |
| Request transformation | ❌                 | ✅                                     |
| Circuit breaking       | ❌                 | ✅                                     |
| Routing by path        | L7 only            | ✅                                     |
| Overhead               | Low                | Higher                                 |

> Often used together: LB in front of API Gateway for the gateway's own HA.

---

## ⚖️ API Gateway vs Reverse Proxy

| Feature        | Reverse Proxy                  | API Gateway                     |
| -------------- | ------------------------------ | ------------------------------- |
| Purpose        | Forward requests, hide servers | Orchestrate microservices       |
| Auth           | ❌                             | ✅                              |
| Business logic | Minimal                        | Yes (routing rules, transforms) |
| Example        | Nginx, HAProxy                 | Kong, AWS API Gateway           |

---

## 🌍 Popular API Gateway Products

| Product                  | Type        | Best For                          |
| ------------------------ | ----------- | --------------------------------- |
| **Kong**                 | Open source | Self-hosted, plugin ecosystem     |
| **AWS API Gateway**      | Managed     | AWS ecosystem, serverless         |
| **Nginx**                | Open source | Can be configured as gateway      |
| **Traefik**              | Open source | Docker/Kubernetes native          |
| **Apigee**               | Enterprise  | Google Cloud, enterprise features |
| **Azure API Management** | Managed     | Azure ecosystem                   |

---

## ⚠️ Gateway Pitfalls

| Pitfall                   | Impact                                | Fix                                          |
| ------------------------- | ------------------------------------- | -------------------------------------------- |
| Gateway is SPOF           | If gateway dies, everything dies      | Run multiple instances behind LB             |
| Too much logic in gateway | Gateway becomes a monolith            | Keep it thin — only cross-cutting concerns   |
| High latency              | Every request passes through gateway  | Optimize, cache at gateway, keep logic light |
| Tight coupling            | Gateway knows too much about services | Use routing rules, not business logic        |

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/api-gateway.excalidraw`](../diagrams/api-gateway.excalidraw)

The diagram shows:

- Client → Gateway → multiple microservices
- Auth, Rate Limiting, Logging modules inside gateway
- Gateway cluster (multiple instances) behind a load balancer
- Circuit breaker to a failing service

---

## 🔑 Key Takeaways

- API Gateway = **single entry point** + cross-cutting concerns
- Move auth, rate limiting, logging **out of services** and into the gateway
- The gateway can become a bottleneck/SPOF — make it HA and keep it lightweight
- Don't put business logic in the gateway — that belongs in services

---

## 🔗 Related Topics

- [Load Balancers](./load-balancers.md)
- [Reverse Proxy](./reverse-proxy.md)
- [Microservices](../07-microservices/microservices.md)
- [Rate Limiting](../08-reliability-and-performance/rate-limiting.md)

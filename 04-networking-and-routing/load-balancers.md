# ⚖️ Load Balancers

> **One-liner:** A load balancer distributes incoming traffic across multiple servers so no single server becomes a bottleneck.

---

## 📌 Why Load Balancers?

```
Without:  [1000 req/sec] → [Server A] 🔥 (overloaded)
                         → [Server B] 💤 (idle)

With LB:  [1000 req/sec] → [Load Balancer]
                               ├──500 req/sec──► [Server A]
                               └──500 req/sec──► [Server B]
```

Benefits:

- Prevents any one server from being overwhelmed
- Enables horizontal scaling
- Provides high availability — removes dead servers from rotation
- SSL termination — offload TLS overhead from app servers

---

## 🔀 Load Balancing Algorithms

### 1. Round Robin

```
Request 1 → Server A
Request 2 → Server B
Request 3 → Server C
Request 4 → Server A (cycles back)
```

✅ Simple, even distribution
❌ Doesn't account for server load or request weight

### 2. Weighted Round Robin

```
Server A (weight 3): gets 3 requests
Server B (weight 1): gets 1 request
Server C (weight 2): gets 2 requests
```

✅ Handles heterogeneous servers
❌ Still doesn't reflect real-time load

### 3. Least Connections

```
Server A: 100 active connections
Server B: 20 active connections  ← send here
Server C: 80 active connections
```

✅ Dynamic — adapts to actual load
✅ Great for long-lived connections (WebSockets)

### 4. IP Hash (Sticky Sessions)

```
hash(client_IP) % numServers → always same server
```

✅ Session affinity — user always hits same server
❌ Defeats purpose of stateless horizontal scaling
❌ Uneven if many users share same IP (corporate NAT)

### 5. Least Response Time

Routes to the server with the lowest combination of active connections + response time.

### 6. Random with Two Choices (Power of Two)

Pick 2 random servers, send to the one with fewer connections.
✅ Surprisingly close to optimal with very low overhead

---

## 🏗️ Layer 4 vs Layer 7 Load Balancers

### Layer 4 (Transport Layer)

- Works with **TCP/UDP** (IP addresses + ports)
- Doesn't inspect packet content
- Very fast, low overhead
- Can't route based on URL path or headers

```
Sees: IP:port → forward to server
```

### Layer 7 (Application Layer)

- Works with **HTTP/HTTPS** content
- Can route based on URL, headers, cookies
- SSL termination here
- Slower (more processing) but far more powerful

```
/api/*  → API servers
/static/* → Static file servers
/ws/*   → WebSocket servers
```

**Example: Nginx as L7 LB**

```nginx
upstream api_servers {
    least_conn;
    server 10.0.0.1:3000;
    server 10.0.0.2:3000;
    server 10.0.0.3:3000;
}

server {
    location /api/ { proxy_pass http://api_servers; }
    location /static/ { proxy_pass http://cdn_server; }
}
```

---

## 💓 Health Checks

LB constantly checks if servers are alive:

```
Passive: Monitor responses — if 5xx for 3 consecutive requests → mark unhealthy
Active:  Send GET /health every 10 seconds → expect 200 OK within 2 seconds
```

When a server is unhealthy → LB stops sending traffic. When it recovers → gradually re-added.

---

## 🔒 SSL Termination

```
Client ──[HTTPS]──► Load Balancer ──[HTTP]──► App Servers
         (encrypted)              (unencrypted, internal network)
```

Benefits:

- App servers don't need SSL certificates
- Reduces CPU overhead on app servers
- Centralized certificate management

---

## ⚠️ Load Balancer as SPOF

The LB itself can become a SPOF!

Solution: **Active-Passive LB Pair**

```
[LB Primary] ──── health ────► [LB Secondary]
       │                              │
  (active)                    (standby — takes over if primary fails)
       └──────────────────────────────┘
                    VIP (Virtual IP floats between them)
```

---

## 🌍 Real Products

| Product        | Type           | Use                                |
| -------------- | -------------- | ---------------------------------- |
| **Nginx**      | Software L4/L7 | Self-hosted, versatile             |
| **HAProxy**    | Software L4/L7 | High-performance, battle-tested    |
| **AWS ALB**    | Managed L7     | AWS, auto-scaling integration      |
| **AWS NLB**    | Managed L4     | Ultra-high performance, static IPs |
| **Cloudflare** | DNS + L7       | Global, DDoS protection            |

---

## 🔑 Key Takeaways

- LBs enable horizontal scaling and eliminate single points of failure
- **L7 LBs** are more powerful but heavier; **L4 LBs** are faster
- **Least connections** is usually the best algorithm for dynamic workloads
- The LB itself needs to be made HA (active-passive pair)

---

## 🔗 Related Topics

- [Horizontal Scaling](../02-scaling/horizontal-scaling.md)
- [API Gateway](./api-gateway.md)
- [Reverse Proxy](./reverse-proxy.md)
- [Rate Limiting](../08-reliability-and-performance/rate-limiting.md)

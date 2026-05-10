# 🗺️ Service Discovery

> **One-liner:** Service discovery is how microservices find each other at runtime — instead of hardcoding IP addresses, services register themselves and clients look them up dynamically.

> 🎨 **Diagram:** [service-discovery.excalidraw](../diagrams/service-discovery.excalidraw) — open in [Excalidraw](https://excalidraw.com) (File → Open)

---

## ❓ The Problem

In microservices, services run on dynamic infrastructure:

- Container IPs change on every restart
- Services scale up/down constantly
- Multiple instances run behind a load balancer
- New versions deploy and old ones disappear

**Hardcoding IPs breaks instantly.** You need a phone book that updates itself.

---

## 🏗️ Two Patterns

### 1. Client-Side Discovery

The client asks a **Service Registry** directly, then calls the service.

```
Client  ──► Registry: "Where is payment-service?"
        ◄── Registry: "192.168.1.5:3001, 192.168.1.6:3001"
Client  ──► payment-service:3001 (client picks one, does LB itself)
```

**Tools:** Eureka (Netflix), Consul

**Pros:** Client controls load balancing logic  
**Cons:** Client must have discovery library for every language

---

### 2. Server-Side Discovery

The client calls a **Load Balancer / API Gateway**, which does the lookup.

```
Client  ──► Load Balancer: "POST /payments"
LB      ──► Registry: "Where is payment-service?"
        ◄── Registry: "192.168.1.5:3001"
LB      ──► payment-service:3001
        ◄── Response
Client  ◄── Response
```

**Tools:** AWS ALB + ECS, Kubernetes, Nginx + Consul

**Pros:** Client is dumb (no discovery code needed)  
**Cons:** Extra hop, LB is a potential bottleneck

---

## 📋 Service Registry

The central database of "who's alive and where."

### Registration (how services join)

```
# Self-registration: service registers itself on startup
POST /v1/agent/service/register
{
  "Name": "payment-service",
  "ID": "payment-service-1",
  "Address": "192.168.1.5",
  "Port": 3001,
  "Check": {
    "HTTP": "http://192.168.1.5:3001/health",
    "Interval": "10s"   ← health check every 10 seconds
  }
}

# Third-party registration: orchestrator registers (Kubernetes does this)
```

### Health Checks

Registry pings each registered service. If it fails N times → marked unhealthy → removed from discovery results.

---

## 🔧 Tools Comparison

| Tool                 | Type                   | Best For                  |
| -------------------- | ---------------------- | ------------------------- |
| **Consul**           | Client-side + KV store | Multi-DC, service mesh    |
| **Eureka** (Netflix) | Client-side            | Spring Boot microservices |
| **etcd**             | Key-value + watch      | Kubernetes backbone       |
| **Kubernetes DNS**   | Server-side            | K8s native (just use it)  |
| **AWS Cloud Map**    | Server-side            | AWS ECS/EKS               |
| **Zookeeper**        | Client-side            | Older Kafka setups        |

### Kubernetes DNS (simplest)

In K8s, services get a DNS name automatically:

```
payment-service.default.svc.cluster.local
# format: <service-name>.<namespace>.svc.cluster.local
```

Just call `http://payment-service/pay` — K8s handles the rest.

---

## ✅ Pros

- No hardcoded IPs — services survive restarts and scaling
- Automatic health check integration
- Enables zero-downtime deployments
- Foundation for service mesh (Istio, Linkerd)

## ❌ Cons

- Registry itself is a single point of failure (needs HA)
- Adds operational complexity
- Stale entries possible if TTL/health checks are misconfigured
- Client-side pattern requires polyglot library support

## ⚖️ When to Use / When NOT to Use

**✅ Use when:**

- Running 5+ microservices that need to communicate
- Containers restart with changing IPs
- Auto-scaling groups where instance count changes
- Multi-region deployments

**❌ Avoid / not needed when:**

- Monolith or 2-3 services with stable IPs
- Already on Kubernetes (use built-in DNS — no extra tool needed)
- Services communicate only through a message queue (no direct calls)

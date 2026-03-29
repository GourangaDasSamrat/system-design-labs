# 🌐 DNS — Domain Name System

> **One-liner:** DNS is the internet's phone book — it translates human-readable domain names (google.com) into machine-readable IP addresses (142.250.195.78).

---

## 📌 Why DNS Exists

Computers communicate using **IP addresses** (numbers), but humans remember **names**. DNS bridges this gap.

Without DNS, you'd have to type `142.250.195.78` instead of `google.com`.

---

## 🔍 DNS Resolution — Step by Step

```
You type: www.google.com
```

1. **Browser Cache** — Did I look this up recently? (TTL-based)
2. **OS Cache / hosts file** — `/etc/hosts` on Linux/Mac
3. **Recursive Resolver** (your ISP or 8.8.8.8) — The middleman
4. **Root Name Server** — Knows who handles `.com`
5. **TLD Name Server** — Knows who handles `google.com`
6. **Authoritative Name Server** — Returns the final IP address
7. Response cached at each level with a **TTL**

---

## 🗂️ DNS Record Types

| Record    | Purpose                            | Example                        |
| --------- | ---------------------------------- | ------------------------------ |
| **A**     | Domain → IPv4 address              | `google.com → 142.250.195.78`  |
| **AAAA**  | Domain → IPv6 address              | `google.com → 2607:f8b0::...`  |
| **CNAME** | Alias → another domain             | `www.google.com → google.com`  |
| **MX**    | Mail server for domain             | `google.com → smtp.google.com` |
| **NS**    | Name server for domain             | `google.com → ns1.google.com`  |
| **TXT**   | Arbitrary text (SPF, verification) | `v=spf1 include:...`           |
| **PTR**   | Reverse lookup (IP → domain)       | `142.250.195.78 → google.com`  |
| **SOA**   | Start of Authority — zone metadata | Serial, TTL defaults           |

---

## ⏱️ TTL (Time to Live)

TTL controls how long a DNS record is **cached** before being re-fetched.

| TTL              | Use Case                           |
| ---------------- | ---------------------------------- |
| `60s`            | Frequent changes (A/B deployments) |
| `300s`           | Default for most records           |
| `86400s` (1 day) | Stable records (MX, NS)            |

**⚠️ Before migrating servers: lower TTL to 60s first, wait for propagation, then change IP.**

---

## 🌍 DNS Hierarchy

```
               Root (.)
              /    |    \
          .com   .org   .io
          /
      google.com
      /         \
  www           mail
```

---

## 🔄 DNS Caching Layers

```
Browser Cache (seconds-minutes)
    ↓ miss
OS / hosts file
    ↓ miss
Recursive Resolver (ISP / 8.8.8.8)  ← caches for TTL
    ↓ miss
Root → TLD → Authoritative
```

---

## 🛡️ DNS Security

| Threat            | Description                              | Defense                                |
| ----------------- | ---------------------------------------- | -------------------------------------- |
| **DNS Spoofing**  | Attacker poisons cache with fake records | DNSSEC                                 |
| **DNS Hijacking** | ISP/government redirects queries         | Use encrypted DNS (DoH, DoT)           |
| **DDoS on DNS**   | Flood resolver to take down domains      | Anycast DNS (Cloudflare, AWS Route 53) |

- **DNSSEC** — Cryptographically signs DNS records
- **DoH (DNS over HTTPS)** — Encrypts DNS queries
- **DoT (DNS over TLS)** — Alternative encryption standard

---

## ⚡ DNS in System Design

### Load Balancing via DNS

```
api.myapp.com  →  [DNS Round Robin]
                  → 10.0.0.1
                  → 10.0.0.2
                  → 10.0.0.3
```

Simple but no health checks — if a server dies, DNS still routes to it until TTL expires.

### GeoDNS

Different IPs returned based on user's location:

```
api.myapp.com from India → 13.235.x.x (Mumbai)
api.myapp.com from US   → 54.144.x.x (Virginia)
```

### Failover via DNS

Change A record to backup server when primary fails. But limited by TTL propagation delay.

---

## 🏢 Popular DNS Providers

| Provider                 | Feature                             |
| ------------------------ | ----------------------------------- |
| **Cloudflare** (1.1.1.1) | Fast, privacy-focused, free         |
| **Google** (8.8.8.8)     | Reliable, global                    |
| **AWS Route 53**         | Deep AWS integration, health checks |
| **Azure DNS**            | Azure ecosystem                     |

---

## 🔑 Key Takeaways

- DNS resolution is **hierarchical and cached**
- Lower TTL before changing server IPs to avoid downtime
- DNS alone is a bad load balancer (no health checks)
- Modern DNS services (Cloudflare, Route 53) offer much more than just name resolution

---

## 🔗 Related Topics

- [Client-Server Model](./client-server-model.md)
- [Load Balancers](../04-networking-and-routing/load-balancers.md)
- [CDN](../04-networking-and-routing/cdn.md)

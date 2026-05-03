# 🛡️ DDoS Protection & Rate Limiting Abuse Cases

> **One-liner:** A DDoS (Distributed Denial of Service) attack overwhelms your system with fake traffic until real users can't get through. Defense is layered — no single solution works alone.

> 🎨 **Diagram:** [ddos-protection.excalidraw](../diagrams/ddos-protection.excalidraw) — open in [Excalidraw](https://excalidraw.com) (File → Open)

---

## ❓ Types of Attacks

| Type                    | How                                 | Target          |
| ----------------------- | ----------------------------------- | --------------- |
| **Volumetric**          | Flood bandwidth (Gbps UDP)          | Network layer   |
| **Protocol**            | Exhaust TCP connections (SYN flood) | Transport layer |
| **Application (L7)**    | HTTP flood, slowloris               | Your app        |
| **Credential stuffing** | Try billions of stolen passwords    | Auth endpoints  |
| **Scraping**            | Extract all your data               | Business logic  |
| **Account enumeration** | Guess valid emails                  | User existence  |

---

## 🧱 Defense in Depth (Layers)

```
Internet
   │
   ▼
┌──────────────────────────────────┐
│  CDN / DDoS Scrubbing Center     │  ← Cloudflare, AWS Shield
│  Filters volumetric attacks      │    Absorbs Tbps-scale traffic
│  IP reputation, anycast routing  │
└──────────────────┬───────────────┘
                   │ (clean traffic only)
                   ▼
┌──────────────────────────────────┐
│  WAF (Web Application Firewall)  │  ← Cloudflare WAF, AWS WAF
│  Blocks SQLi, XSS, bad patterns  │    Rate limits by IP/user-agent
│  OWASP Top 10 rules              │    Bot fingerprinting
└──────────────────┬───────────────┘
                   │
                   ▼
┌──────────────────────────────────┐
│  API Gateway / Load Balancer     │  ← Rate limiting per API key
│  Request throttling              │    IP allowlist/blocklist
│  Auth enforcement                │    Quota per user tier
└──────────────────┬───────────────┘
                   │
                   ▼
┌──────────────────────────────────┐
│  Application Rate Limiting       │  ← Redis-backed counters
│  Per-user, per-endpoint limits   │    Sliding window algorithm
│  CAPTCHA triggers                │    Business logic rules
└──────────────────┬───────────────┘
                   │
                   ▼
            Your Services
```

---

## 🔑 Rate Limiting Abuse Cases

### 1. Credential Stuffing Defense

```javascript
// Stricter limits on auth endpoints
rateLimiter({
  "/api/login": { max: 5, window: "15min", by: "ip" },
  "/api/forgot-password": { max: 3, window: "1hr", by: "ip" },
  "/api/register": { max: 10, window: "1hr", by: "ip" },
});

// After 5 failures: require CAPTCHA
// After 10 failures: temp block IP for 1 hour
// Alert security team if 1000+ failures from IP range
```

### 2. API Abuse Tiers

```
Free tier:     100 req/day, 10 req/min
Pro tier:      10,000 req/day, 100 req/min
Enterprise:    Unlimited, custom limits

// Headers to return (industry standard):
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 1699999999  (unix timestamp)
Retry-After: 60               (when limit hit, return 429)
```

### 3. Slow Loris Defense

Attacker sends HTTP headers very slowly to hold connections open.

```nginx
# Nginx settings
client_body_timeout 10s;
client_header_timeout 10s;
keepalive_timeout 5s 5s;
send_timeout 10s;
```

---

## 🤖 Bot Detection Signals

| Signal               | Legitimate User | Bot                   |
| -------------------- | --------------- | --------------------- |
| Request rate         | ~1-2 req/sec    | 100s req/sec          |
| User-agent           | Chrome/Firefox  | Empty or spoofed      |
| TLS fingerprint      | Real browser    | curl, Python requests |
| Behavioral pattern   | Random, varied  | Uniform, sequential   |
| JavaScript execution | Yes             | No (headless bots)    |
| IP reputation        | Clean           | Known datacenter/VPN  |

**Cloudflare Bot Management** and **reCAPTCHA v3** automate most of this.

---

## ⚙️ AWS Shield Tiers

| Tier                | Protection                                    | Cost         |
| ------------------- | --------------------------------------------- | ------------ |
| **Shield Standard** | Automatic L3/L4 protection                    | Free         |
| **Shield Advanced** | L7 protection, DDoS cost protection, 24/7 DRT | $3,000/month |

Most startups: Cloudflare Free or Pro tier is sufficient.

---

## ✅ Pros

- CDN/scrubbing centers absorb traffic before it reaches you
- WAF handles OWASP Top 10 without code changes
- Rate limiting protects business logic and database
- Layered approach means no single point of failure

## ❌ Cons

- Cloudflare/Shield costs money at scale
- Sophisticated L7 attacks (mimic real users) are hard to distinguish
- Blocking legitimate users with aggressive rate limits hurts UX
- Misconfigured WAF rules cause false positives

## ⚖️ When to Use / When NOT to Use

**✅ Use when:**

- Any public-facing API or website (DDoS is a real threat)
- Financial or health apps (credential stuffing target)
- APIs with expensive backend operations (LLM calls, DB writes)
- Any endpoint that is unauthenticated

**❌ Don't over-engineer when:**

- Internal-only API (behind VPN/private network)
- Early prototype with 10 users — Cloudflare Free is enough
- Traffic is already behind authentication (harder to abuse)

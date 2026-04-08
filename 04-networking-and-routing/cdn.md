# 🌍 CDN — Content Delivery Network

> **One-liner:** A globally distributed network of servers that caches your static content close to users, dramatically reducing latency and offloading traffic from your origin server.

---

## 📌 The Problem

```
User in Mumbai → requests image from origin server in US-East
Latency: 200–300ms (round trip across the globe)

Multiply by: every image, CSS file, JS bundle, video chunk...
Result: Slow, painful experience
```

---

## 💡 The Solution: CDN

```
User in Mumbai → requests image
CDN Edge Node in Mumbai → serves cached image
Latency: 5–20ms (local)
```

CDN brings content **closer to users** by caching it at edge locations worldwide.

---

## 🗺️ CDN Architecture

```
                        [Origin Server - US]
                               │
              ┌────────────────┼────────────────┐
              │                │                │
        [Edge - Mumbai]  [Edge - London]  [Edge - Singapore]
              │
         [User - India] ← 15ms →
```

Cloudflare has 300+ edge locations. AWS CloudFront has 400+ PoPs (Points of Presence).

---

## 📦 What CDNs Cache

### Static Assets (Always CDN-appropriate)

- Images (JPEG, PNG, WebP, SVG)
- JavaScript bundles (`bundle.js`)
- CSS files (`styles.css`)
- Fonts (`.woff2`, `.ttf`)
- Videos (HLS/DASH streaming chunks)
- HTML (for static sites)
- PDF documents

### Dynamic Content (Advanced CDN features)

- API responses (short TTL — 1–5 minutes)
- Personalized content (with Vary headers)
- HTML with Edge Side Includes (ESI)

---

## 🔄 Cache Flow: CDN Request Lifecycle

### Cache HIT (fast path)

```
User → CDN Edge → Cache HIT → Serve from edge
```

### Cache MISS (slow path — first request)

```
User → CDN Edge → Cache MISS
               → Fetch from Origin Server
               → Store in edge cache
               → Serve to user
```

### Cache INVALIDATION (when you deploy)

```
You push new bundle.js →
Signal CDN to invalidate old bundle.js →
Next request → MISS → fetches new version → caches it
```

---

## ⏱️ Cache Control Headers

The origin server controls CDN caching via HTTP headers:

```http
# Cache for 1 year (immutable assets with content hash in filename)
Cache-Control: public, max-age=31536000, immutable

# Cache for 5 minutes (dynamic-ish content)
Cache-Control: public, max-age=300

# Don't cache (private user data)
Cache-Control: private, no-store

# Cache but validate with server first
Cache-Control: no-cache
ETag: "abc123"
```

### Filename Hashing Strategy

```
bundle.js       → hard to cache long (changes on deploy)
bundle.a3f4c1.js → cache for 1 year (hash changes on deploy)
```

---

## 🔒 CDN Security Features

| Feature             | Description                                                        |
| ------------------- | ------------------------------------------------------------------ |
| **DDoS Protection** | Absorb volumetric attacks at edge (Cloudflare: 150+ Tbps capacity) |
| **WAF**             | Web Application Firewall — block SQLi, XSS at edge                 |
| **Bot Protection**  | Detect and block malicious bots                                    |
| **TLS Termination** | Handle HTTPS at edge globally                                      |
| **Rate Limiting**   | Throttle requests at edge before they hit origin                   |
| **Geo-blocking**    | Block traffic from specific countries                              |

---

## ⚙️ CDN Configuration Example (Cloudflare)

```
Page Rules:
*.myapp.com/static/*  → Cache Everything, Edge TTL: 1 year
*.myapp.com/api/*     → Bypass Cache (dynamic)
*.myapp.com/*.html    → Cache, Edge TTL: 1 hour
```

---

## 🔄 Push vs Pull CDN

### Pull CDN (Most Common)

- Origin server remains the source of truth
- CDN fetches content from origin **on first request** (cache miss)
- Automatic — no manual upload needed
- Examples: Cloudflare, CloudFront

### Push CDN

- You explicitly **upload** content to CDN
- CDN serves it from edge; origin doesn't need to exist for every request
- Good for known, large files (video, software downloads)
- Examples: AWS S3 + CloudFront for uploads, Akamai NetStorage

---

## 🌍 Popular CDN Providers

| Provider           | Strength                                   |
| ------------------ | ------------------------------------------ |
| **Cloudflare**     | Best DDoS protection, free tier, 300+ PoPs |
| **AWS CloudFront** | Deep AWS integration, Lambda@Edge          |
| **Fastly**         | Real-time purging, developer-friendly      |
| **Akamai**         | Largest network, enterprise                |
| **BunnyCDN**       | Cheap, fast, developer-friendly            |

---

## 📊 CDN Impact on Performance

Without CDN (Mumbai user, US origin):

- Latency: ~200ms
- Origin bandwidth: 100GB/day

With CDN:

- Latency: ~15ms (13× faster!)
- Cache hit ratio: ~85%
- Origin bandwidth: ~15GB/day (85% reduction)

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/cdn.excalidraw`](../diagrams/cdn.excalidraw)

The diagram shows:

- World map with edge nodes
- Cache HIT vs MISS flows
- Origin server → edge propagation
- Cache-Control header on HTTP response

---

## 🔑 Key Takeaways

- CDN = **geographic caching** — serve content from the closest edge node
- Reduces latency, reduces origin load, handles DDoS
- Use long TTLs + **content hashing** for static assets
- Most modern apps should use a CDN from day 1 (Cloudflare free tier is excellent)

---

## 🔗 Related Topics

- [Caching Strategies](../05-caching/caching-strategies.md)
- [Load Balancers](./load-balancers.md)
- [DNS](../01-fundamentals/dns.md)
- [Rate Limiting](../08-reliability-and-performance/rate-limiting.md)

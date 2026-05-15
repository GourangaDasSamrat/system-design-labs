# ⚡ Edge Computing

> CDN serves static files. Edge computing runs **logic** at the edge. This is the next level.

---

## CDN vs Edge Computing

|                     | CDN                                    | Edge Computing                               |
| ------------------- | -------------------------------------- | -------------------------------------------- |
| **What it does**    | Caches static files (HTML, JS, images) | Runs code at edge nodes                      |
| **Dynamic content** | ❌ Must go to origin                   | ✅ Can compute at edge                       |
| **Latency**         | Low for cache hits                     | Low even for dynamic responses               |
| **Examples**        | CloudFront, Fastly                     | Cloudflare Workers, Lambda@Edge, Deno Deploy |

---

## 🧠 Edge Functions

Code that runs in the CDN network, geographically close to users.

**Runtimes:**

- **Cloudflare Workers** — V8 isolates, JS/WASM, 0ms cold start
- **Lambda@Edge** — AWS, triggered by CloudFront, higher cold starts
- **Vercel Edge Functions** — built on Cloudflare Workers
- **Deno Deploy** — TypeScript-native

**Constraints:**

- No filesystem access
- Limited CPU time (1–50ms budgets)
- Limited memory (~128MB)
- No long-running processes
- Some have no Node.js APIs (use Web APIs only)

---

## 🔑 Use Cases

### 1. Authentication & Authorization at the Edge

```
User → Edge Node → check JWT → valid? → forward to origin
                              invalid? → return 401 immediately
```

- Block unauthenticated requests before they hit your servers
- No round-trip to origin for auth validation
- Reduces load on origin by rejecting bad requests early

### 2. Personalization

```
User request hits edge →
  Read cookie / geo / device header →
  Rewrite URL or inject headers →
  Serve personalized variant
```

- A/B testing without origin involvement
- Locale-based content (language, currency)
- Device-based content (mobile vs desktop)

### 3. Bot Detection & Rate Limiting

- Block bots at the edge before they consume origin resources
- Cloudflare's Bot Management runs entirely at edge

### 4. Request/Response Transformation

- Add security headers (CSP, HSTS) to every response
- Strip sensitive headers before forwarding
- Rewrite URLs for legacy compatibility

### 5. Geo-Based Redirects

```javascript
// Cloudflare Worker example
export default {
  async fetch(request) {
    const country = request.cf.country;
    if (country === "DE") {
      return Response.redirect("https://de.example.com" + request.url);
    }
    return fetch(request);
  },
};
```

---

## 🗺️ Edge vs Origin Decision Tree

```
Is the response the same for all users?
  YES → CDN cache it (static)
  NO  → Does personalization need server state/DB?
          NO  → Edge function (compute from request headers/cookies)
          YES → Origin server (but edge can cache origin response per variant)
```

---

## 📦 Edge KV Storage

Some edge platforms offer distributed key-value stores accessible from edge:

- **Cloudflare KV** — globally replicated, eventual consistency, low-latency reads
- **Cloudflare Durable Objects** — strongly consistent, single-region, great for coordination

Use for:

- Feature flags
- Rate limit counters (approximate)
- Session data
- Config that changes infrequently

---

## ⚖️ Trade-offs

✅ **Pros**

- Sub-10ms response for many use cases
- Reduces load and cost on origin servers
- No infrastructure to manage (serverless)
- Global presence instantly

❌ **Cons**

- No access to databases or internal services directly
- Limited runtime environment (no full Node.js)
- Debugging is harder (distributed, remote)
- Cold start exists (minimal but nonzero for Lambda@Edge)
- Vendor lock-in (Cloudflare Workers API ≠ Lambda@Edge API)

⚖️ **When to use**

- Auth validation, bot blocking, geo-redirects
- A/B testing, personalization headers
- Performance-critical responses that don't need DB

**When NOT to use**

- Complex business logic requiring DB queries
- Long-running operations
- When you need full OS / filesystem access

---

## 🎨 Diagram

See: [`diagrams/edge-computing.excalidraw`](../diagrams/edge-computing.excalidraw)

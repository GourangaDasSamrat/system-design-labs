# 📦 API Versioning

> **One-liner:** API versioning lets you evolve your API without breaking existing clients — you maintain multiple contract versions simultaneously until clients migrate.

> 🎨 **Diagram:** [api-versioning.excalidraw](../diagrams/api-versioning.excalidraw) — open in [Excalidraw](https://excalidraw.com) (File → Open)

---

## ❓ Why Version Your API?

Once a public API has consumers, you can't change it freely:

- Renaming a field breaks every client using the old name
- Removing an endpoint causes 404s for clients that depend on it
- Changing response format silently corrupts client logic

Versioning gives clients time to migrate at their own pace.

---

## 🏗️ 4 Versioning Strategies

### 1. URI Path Versioning ✅ (Most Common)

```
GET /api/v1/users/42
GET /api/v2/users/42   ← new version with different response shape
```

**Pros:** Obvious, easy to route, cacheable by CDN  
**Cons:** Pollutes URLs, clients must change URLs to upgrade

---

### 2. Query Parameter Versioning

```
GET /api/users/42?version=1
GET /api/users/42?version=2
```

**Pros:** URL stays stable  
**Cons:** Easy to forget, doesn't work well with caching

---

### 3. Header Versioning

```
GET /api/users/42
Accept: application/vnd.myapi.v2+json
              OR
X-API-Version: 2
```

**Pros:** Clean URLs, aligns with REST purists  
**Cons:** Not visible in browser, harder to test/share URLs

---

### 4. Content Negotiation (Accept header)

```
GET /api/users/42
Accept: application/vnd.company.user-v2+json
```

Used by GitHub API and Stripe. Powerful but complex.

---

## 🔀 Routing Versions

At the API Gateway or router level:

```javascript
// Express routing example
const v1Router = require('./routes/v1');
const v2Router = require('./routes/v2');

app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);

// Or via API Gateway (AWS API Gateway, Kong, Nginx):
# nginx.conf
location /api/v1/ { proxy_pass http://service-v1/; }
location /api/v2/ { proxy_pass http://service-v2/; }
```

---

## 📋 Breaking vs Non-Breaking Changes

| Change                             | Breaking? | Strategy                             |
| ---------------------------------- | --------- | ------------------------------------ |
| Add new optional field to response | ❌ No     | Safe — clients ignore unknown fields |
| Add new optional request param     | ❌ No     | Safe — use defaults for old clients  |
| Remove a field                     | ✅ Yes    | New version required                 |
| Rename a field                     | ✅ Yes    | New version required                 |
| Change field type (string→int)     | ✅ Yes    | New version required                 |
| Change HTTP method                 | ✅ Yes    | New version required                 |
| Add required request param         | ✅ Yes    | New version required                 |

**Rule:** If an existing client would break without code changes, it's breaking.

---

## 🔄 Migration Lifecycle

```
v1 released
    ↓
v2 released (both live simultaneously)
    ↓
v1 deprecated — notice given (3-6 months typical)
    ↓
v1 sunset — clients that haven't migrated break
    ↓
v1 removed from infrastructure
```

**Deprecation headers** (tell clients they're on old version):

```
HTTP/1.1 200 OK
Deprecation: true
Sunset: Sat, 01 Jan 2026 00:00:00 GMT
Link: <https://api.example.com/v2/users>; rel="successor-version"
```

---

## 🏷️ Semantic Versioning for Internal APIs

```
MAJOR.MINOR.PATCH

v2.0.0 → breaking change (new MAJOR version)
v2.1.0 → non-breaking additions (new MINOR version)
v2.1.1 → bug fix (PATCH)
```

For public REST APIs, only expose MAJOR version in the URL (v1, v2). Minor/patch changes are transparent.

---

## 🗂️ Version Coexistence Patterns

### Shared Code, Version-Specific Transform

```javascript
// Core business logic shared
const userService = require("./services/userService");

// v1 controller — old response shape
router.get("/v1/users/:id", async (req, res) => {
  const user = await userService.getUser(req.params.id);
  res.json({ user_id: user.id, user_name: user.name }); // v1 shape
});

// v2 controller — new response shape
router.get("/v2/users/:id", async (req, res) => {
  const user = await userService.getUser(req.params.id);
  res.json({ id: user.id, name: user.name, email: user.email }); // v2 shape
});
```

---

## ✅ Pros

- Clients can migrate at their own pace
- Enables continuous API evolution without coordination
- Clear contracts per version — easy to document
- URI versioning is CDN-cacheable and easy to debug

## ❌ Cons

- Maintaining multiple versions increases codebase complexity
- Duplicate code / logic divergence over time
- Clients often stay on old versions longer than expected
- Sunset enforcement requires communicating with all API consumers

## ⚖️ When to Use / When NOT to Use

**✅ Use when:**

- Public APIs with external consumers
- Mobile apps (you can't force users to update immediately)
- B2B APIs where clients control their upgrade schedule
- Any breaking change to a live API

**❌ Avoid / not needed when:**

- Internal APIs where you control all consumers (coordinate and deploy together)
- GraphQL APIs (schema evolution handles this differently)
- Early product with no external consumers yet (iterate freely)

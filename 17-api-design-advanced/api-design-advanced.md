# 🔌 API Design — Advanced

> You know the basics. Here's what separates good API design from great API design.

---

## 📄 Pagination Strategies

Never return unbounded lists. Always paginate.

### 1. Offset-Based Pagination

```
GET /posts?limit=20&offset=40
```

- Simple, easy to implement
- ✅ Jump to any page directly ("go to page 5")
- ❌ **Page drift:** if items are added/deleted between requests, you miss or duplicate items
- ❌ Performance degrades at high offsets (`OFFSET 100000` scans 100k rows and discards them)
- **Use when:** small datasets, admin panels, search results

### 2. Cursor-Based Pagination (Recommended for feeds)

```
GET /posts?limit=20&cursor=eyJpZCI6MTIzfQ==
Response: { data: [...], nextCursor: "eyJpZCI6MTQ0fQ==" }
```

- Cursor = encoded pointer to last item seen (e.g., base64 of `{id: 123}`)
- ✅ Stable — no page drift even if new items inserted
- ✅ Efficient — WHERE id > 123 LIMIT 20 uses index perfectly
- ❌ Can't jump to arbitrary pages
- **Use when:** infinite scroll, Twitter/Instagram feeds, real-time data

### 3. Keyset Pagination

- Variant of cursor — uses actual column values (not encoded)

```
GET /posts?limit=20&after_id=123&after_created_at=2024-01-15
```

- Multi-column sort support
- Works naturally with DB indexes

### 4. Page-Based (Simple)

```
GET /posts?page=3&per_page=20
```

- Like offset but friendlier naming
- Same problems as offset

---

## 🔍 Filtering & Sorting

### Filtering Patterns

```
# Simple equality
GET /users?status=active&role=admin

# Range filters
GET /orders?created_after=2024-01-01&created_before=2024-12-31
GET /products?price_min=10&price_max=100

# Multi-value
GET /products?category=electronics,phones

# Search
GET /users?search=john
```

**DB implications:**

- Each filter = potential WHERE clause
- Ensure composite indexes match common filter combinations
- `status + role` both filtered → composite index `(status, role)`

### Sorting

```
GET /products?sort=price&order=asc
GET /products?sort=-created_at          # prefix - for desc
GET /products?sort=price,-created_at    # multi-sort
```

- Sort key must be indexed (or full table scan)
- For cursor pagination: sort field must be part of the cursor

---

## 📊 GraphQL Basics

REST: client gets what the server decides to send.
GraphQL: client asks for exactly what it needs.

```graphql
# REST: GET /users/123 returns ALL user fields
# GraphQL: client specifies exactly what it wants

query {
  user(id: "123") {
    name
    email
    posts(limit: 3) {
      title
      createdAt
    }
  }
}
```

**Problems GraphQL solves:**

- **Over-fetching:** REST returns 50 fields, you need 3
- **Under-fetching:** Need to call 3 REST endpoints to build one page
- **Type safety:** Schema defines exactly what's available

**When to use GraphQL:**

- Multiple clients (mobile, web, third-party) with different data needs
- Rapidly evolving APIs (add fields without breaking clients)
- Complex, interconnected data (think: social graph)

**When to stick with REST:**

- Simple CRUD APIs
- Public APIs (REST is more universally understood)
- File uploads, streaming (REST handles these better)
- Small team — GraphQL has operational overhead

**N+1 Problem:**

```graphql
query {
  users {
    posts {
      comments
    }
  }
}
# Without DataLoader: 1 + N + N*M queries 🔥
# With DataLoader: batch loads → 3 queries total ✅
```

---

## 🏷️ API Versioning

You mentioned this — here's the full picture:

### URL Versioning (Most common)

```
GET /v1/users
GET /v2/users
```

- ✅ Very explicit, easy to route
- ❌ URL "should" be a resource, not a version

### Header Versioning

```
GET /users
Accept: application/vnd.myapi.v2+json
```

- ✅ Clean URLs
- ❌ Harder to test (can't just paste URL in browser)

### Query Parameter

```
GET /users?version=2
```

- ✅ Easy to test
- ❌ Optional params = easy to forget → inconsistent behavior

### Sunset Policy

- Announce deprecation date for old versions
- Return `Deprecation` and `Sunset` headers on old endpoints
- Give clients 6–12 months to migrate

---

## 🔒 API Best Practices (Interview Checklist)

| Topic         | Best Practice                                                    |
| ------------- | ---------------------------------------------------------------- |
| Auth          | Bearer token in `Authorization` header, not URL                  |
| Errors        | Consistent error format: `{ error: { code, message, details } }` |
| Rate Limiting | Return `Retry-After` header, 429 status                          |
| Idempotency   | POST with `Idempotency-Key` header for safe retries              |
| Status Codes  | Use them correctly: 201 Created, 204 No Content, 409 Conflict    |
| HATEOAS       | Return links to related resources (optional but impressive)      |
| Compression   | gzip responses > 1KB                                             |

---

## ⚖️ Trade-offs

⚖️ **REST vs GraphQL at interview:**

- Default to REST for simplicity
- Propose GraphQL when: multiple client types, complex data graph, bandwidth-sensitive (mobile)

⚖️ **Offset vs Cursor pagination:**

- Default to cursor for production feeds
- Use offset for admin/search (random access needed)

---

## 🎨 Diagram

See: [`diagrams/api-design-advanced.excalidraw`](../diagrams/api-design-advanced.excalidraw)

# 🔒 Authentication & Authorization — JWT & OAuth 2.0

> **One-liner:** Authentication proves _who you are_; Authorization proves _what you're allowed to do_. JWT and OAuth 2.0 are the industry standards for doing both at scale.

> 🎨 **Diagram:** [auth-jwt-oauth.excalidraw](../diagrams/auth-jwt-oauth.excalidraw) — open in [Excalidraw](https://excalidraw.com) (File → Open)

---

## 🎫 JWT — JSON Web Token

A self-contained, signed token that carries claims about the user. No database lookup needed to verify.

### Structure: Header.Payload.Signature

```
eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOjQyLCJyb2xlIjoiYWRtaW4iLCJleHAiOjE3MDAwMDB9.abc123sig

  HEADER              PAYLOAD                                     SIGNATURE
  {                   {                                           HMACSHA256(
    "alg": "HS256",     "userId": 42,                              base64(header) + "." +
    "typ": "JWT"        "role": "admin",                           base64(payload),
  }                     "iat": 1699999999,                         secretKey
                        "exp": 1700003599  ← expires in 1hr        )
                      }
```

### JWT Flow

```
1. Login:    Client ──► POST /login {email, password}
                        Server validates, creates JWT
             Client ◄── { token: "eyJ..." }

2. Use API:  Client ──► GET /profile
                        Headers: Authorization: Bearer eyJ...
                        Server decodes JWT, checks exp, checks role
             Client ◄── { user data }
```

### Stateless = Scalable

Server doesn't store sessions. Any server instance can verify the token using the secret key. No Redis/DB lookup needed per request.

### JWT Risks & Mitigations

| Risk                                 | Mitigation                               |
| ------------------------------------ | ---------------------------------------- |
| Token stolen                         | Short expiry (15min) + Refresh tokens    |
| Can't invalidate before expiry       | Refresh token blacklist in Redis         |
| Payload visible (base64 ≠ encrypted) | Never store sensitive data in payload    |
| Weak secret                          | Use RS256 (asymmetric) for multi-service |

---

## 🔑 OAuth 2.0 — Delegated Authorization

OAuth lets a third party app access resources on your behalf, without sharing your password.

**"Sign in with Google"** is OAuth 2.0 in action.

### Authorization Code Flow (Recommended)

```
User         Client App        Authorization Server       Resource Server
  |               |             (Google/GitHub/etc)          (Your API)
  |──"Login"─────►|
  |               |──Redirect──►|
  |               |  client_id  |
  |◄──────────────|             |
  |────Login to──►|             |
  |   Google      |             |
  |               |◄─auth code──|
  |               |             |
  |               |──POST /token|
  |               |  code +     |
  |               |  client_secret
  |               |◄─access_token + refresh_token
  |               |                                         |
  |               |──GET /userinfo (Bearer access_token)───►|
  |               |◄── { id, email, name }──────────────────|
  |               |
  |◄──Logged in!──|
```

### Key OAuth Concepts

| Term                     | What It Is                                           |
| ------------------------ | ---------------------------------------------------- |
| **Client**               | Your app                                             |
| **Resource Owner**       | The user                                             |
| **Authorization Server** | Google, GitHub, Auth0 (issues tokens)                |
| **Resource Server**      | API that accepts access tokens                       |
| **Access Token**         | Short-lived (1hr) — use to call APIs                 |
| **Refresh Token**        | Long-lived (30 days) — exchange for new access token |
| **Scope**                | Permissions granted: `read:email`, `write:profile`   |

---

## 🔐 Session vs JWT Comparison

|            | **Session**                  | **JWT**                             |
| ---------- | ---------------------------- | ----------------------------------- |
| Storage    | Server-side (Redis/DB)       | Client-side (localStorage / cookie) |
| Revocation | Instant (delete session)     | Must wait for expiry (or blacklist) |
| Scale      | Needs shared session store   | Stateless — any server works        |
| Payload    | Just session ID              | Full claims (userId, role, etc.)    |
| Best for   | Monolith, small-medium scale | Microservices, APIs, mobile         |

---

## ✅ Pros

**JWT:** Stateless, scalable, no DB lookup per request, cross-service  
**OAuth:** No password sharing, granular scopes, industry standard for SSO

## ❌ Cons

**JWT:** Revocation is hard, payload visible, long tokens add overhead  
**OAuth:** Complex flow, misconfig is a security disaster, token leakage risks

## ⚖️ When to Use / When NOT to Use

**✅ JWT — use when:**

- REST APIs serving mobile or SPA clients
- Microservices that need to pass identity between services
- Stateless, horizontally scaled backends

**✅ OAuth — use when:**

- Third-party "Sign in with X" integrations
- Allowing external apps to access user data on your platform
- Enterprise SSO (Single Sign-On)

**❌ Avoid JWT when:**

- You need instant logout / token revocation (use sessions with Redis)
- Highly sensitive systems where payload visibility is a concern

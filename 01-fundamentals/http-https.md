# 🔒 HTTP & HTTPS

> **One-liner:** HTTP is the protocol for transferring data on the web; HTTPS adds TLS encryption so that data can't be snooped or tampered with in transit.

---

## 📌 HTTP — HyperText Transfer Protocol

HTTP is a **stateless, request-response** application-layer protocol. Every interaction is independent — the server has no memory of previous requests by default.

### Request Structure

```
GET /api/users/42 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGci...
Accept: application/json
```

### Response Structure

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600

{ "id": 42, "name": "Rahul" }
```

---

## 🔢 HTTP Methods

| Method    | Purpose                      | Body? | Idempotent? |
| --------- | ---------------------------- | ----- | ----------- |
| `GET`     | Retrieve resource            | ❌    | ✅          |
| `POST`    | Create resource              | ✅    | ❌          |
| `PUT`     | Replace resource entirely    | ✅    | ✅          |
| `PATCH`   | Partially update resource    | ✅    | ❌          |
| `DELETE`  | Remove resource              | ❌    | ✅          |
| `HEAD`    | Like GET but no body         | ❌    | ✅          |
| `OPTIONS` | Ask what methods are allowed | ❌    | ✅          |

**Idempotent** = calling it N times has same effect as calling it once.

---

## 📊 HTTP Status Codes

| Range | Category      | Examples                                                                                         |
| ----- | ------------- | ------------------------------------------------------------------------------------------------ |
| 1xx   | Informational | `100 Continue`                                                                                   |
| 2xx   | Success       | `200 OK`, `201 Created`, `204 No Content`                                                        |
| 3xx   | Redirection   | `301 Moved Permanently`, `304 Not Modified`                                                      |
| 4xx   | Client Error  | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `429 Too Many Requests` |
| 5xx   | Server Error  | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`                        |

---

## 🔄 HTTP Versions

| Version      | Key Feature                                               | Year |
| ------------ | --------------------------------------------------------- | ---- |
| **HTTP/1.0** | New TCP connection per request                            | 1996 |
| **HTTP/1.1** | Keep-alive, persistent connections, pipelining            | 1997 |
| **HTTP/2**   | Multiplexing, header compression (HPACK), binary protocol | 2015 |
| **HTTP/3**   | QUIC protocol (UDP-based), 0-RTT, faster handshake        | 2022 |

### HTTP/1.1 vs HTTP/2 — Multiplexing

```
HTTP/1.1:  REQ1 ──► | wait | ◄── RES1
           REQ2 ──► | wait | ◄── RES2   (sequential)

HTTP/2:    REQ1 ─┐         ┌─ RES1
           REQ2 ─┤─ wire ──┤─ RES2      (parallel, same connection)
           REQ3 ─┘         └─ RES3
```

---

## 🔐 HTTPS — HTTP over TLS

HTTPS = HTTP + **TLS (Transport Layer Security)**

### What TLS Provides

| Property           | Meaning                                                |
| ------------------ | ------------------------------------------------------ |
| **Encryption**     | Data is unreadable to third parties                    |
| **Authentication** | Server proves it's who it says it is (via certificate) |
| **Integrity**      | Data can't be tampered with without detection          |

### TLS Handshake (Simplified)

```
Client                          Server
  │──── ClientHello (TLS ver) ──►│
  │◄─── ServerHello + Cert ──────│
  │── Verify cert with CA ───────│ (checks certificate authority)
  │──── Key Exchange ────────────│
  │◄─── Key Exchange ────────────│
  │   [Symmetric key derived]    │
  │◄════ Encrypted data ═════════│
```

### SSL vs TLS

- **SSL** is deprecated (SSL 2.0, 3.0 — broken)
- **TLS 1.2** is widely used
- **TLS 1.3** is modern (faster handshake, better ciphers)
- When people say "SSL certificate" they mean **TLS certificate**

---

## 🍪 Cookies & Sessions (Handling Statelessness)

Since HTTP is stateless, we use:

| Mechanism    | How it works                                                                      |
| ------------ | --------------------------------------------------------------------------------- |
| **Cookies**  | Server sends `Set-Cookie` header; browser sends it back on every request          |
| **Sessions** | Server stores session data, client stores session ID in cookie                    |
| **JWT**      | Stateless token — server signs a token, client stores it (localStorage or cookie) |

---

## 📡 Long-Polling, SSE, WebSockets

| Pattern                      | Direction                    | Use Case                              |
| ---------------------------- | ---------------------------- | ------------------------------------- |
| **HTTP Polling**             | Client → Server (repeated)   | Simple updates, high latency          |
| **Long Polling**             | Client holds connection open | Chat, notifications                   |
| **SSE (Server-Sent Events)** | Server → Client (one way)    | Live scores, feeds                    |
| **WebSocket**                | Bidirectional                | Chat apps, multiplayer games, trading |

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/http-tls-handshake.excalidraw`](../diagrams/http-tls-handshake.excalidraw)

---

## 🔑 Key Takeaways

- HTTP is **stateless** — use cookies/JWT to add state
- Always use **HTTPS** in production (free certs via Let's Encrypt)
- HTTP/2 gives huge performance gains via multiplexing
- Understand status codes cold — interviewers ask about 401 vs 403, 502 vs 503, etc.

---

## 🔗 Related Topics

- [Client-Server Model](./client-server-model.md)
- [API Gateway](../04-networking-and-routing/api-gateway.md)
- [Rate Limiting](../08-reliability-and-performance/rate-limiting.md)
- [Caching Strategies](../05-caching/caching-strategies.md)

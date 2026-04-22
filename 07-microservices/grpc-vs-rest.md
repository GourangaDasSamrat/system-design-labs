# 🔀 Service-to-Service Communication — gRPC vs REST

> **One-liner:** REST uses JSON over HTTP/1.1 and is human-readable; gRPC uses binary Protocol Buffers over HTTP/2 and is 5-10× faster — pick based on who's calling your service.

> 🎨 **Diagram:** [grpc-vs-rest.excalidraw](../diagrams/grpc-vs-rest.excalidraw) — open in [Excalidraw](https://excalidraw.com) (File → Open)

---

## 📡 REST — Representational State Transfer

The universal language of the web. Every language, framework, and tool speaks it.

```
POST /api/users HTTP/1.1
Host: user-service
Content-Type: application/json

{
  "name": "Arif",
  "email": "arif@example.com"
}

─────────────────────────────

HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 42,
  "name": "Arif",
  "email": "arif@example.com",
  "createdAt": "2025-03-15T10:00:00Z"
}
```

**Best for:** Public APIs, browser clients, external third-party consumers

---

## ⚡ gRPC — Google Remote Procedure Call

Binary protocol using Protocol Buffers (protobuf). Strongly typed contract defined in `.proto` files.

```protobuf
// user.proto — the contract (shared between client and server)
syntax = "proto3";

service UserService {
  rpc CreateUser (CreateUserRequest) returns (CreateUserResponse);
  rpc GetUser    (GetUserRequest)    returns (User);
  rpc ListUsers  (ListRequest)       returns (stream User);  // server streaming
}

message CreateUserRequest {
  string name  = 1;
  string email = 2;
}

message User {
  int32  id    = 1;
  string name  = 2;
  string email = 3;
}
```

```javascript
// Generated client code (auto-generated from .proto)
const client = new UserServiceClient(channel);
const response = await client.createUser({
  name: "Arif",
  email: "arif@example.com",
});
console.log(response.id); // 42
```

**Best for:** Internal microservice-to-microservice calls

---

## ⚖️ Head-to-Head Comparison

| Feature              | REST                         | gRPC                         |
| -------------------- | ---------------------------- | ---------------------------- |
| **Protocol**         | HTTP/1.1 (or 2)              | HTTP/2 (always)              |
| **Format**           | JSON (text)                  | Protobuf (binary)            |
| **Performance**      | Good                         | ~5-10× faster                |
| **Payload size**     | Larger (verbose JSON)        | Smaller (binary)             |
| **Streaming**        | Workarounds (SSE, WebSocket) | Native bi-directional        |
| **Browser support**  | ✅ Native                    | ❌ Needs gRPC-web proxy      |
| **Schema**           | Optional (OpenAPI)           | Required (.proto)            |
| **Debugging**        | Easy (curl, Postman)         | Harder (binary format)       |
| **Code gen**         | Manual or OpenAPI            | Auto-generated client/server |
| **Language support** | Universal                    | 12+ official languages       |

---

## 🏗️ Hybrid Architecture (Common Pattern)

```
External Clients (browsers, mobile)
         │
         │ REST / JSON  (public API)
         ▼
    API Gateway
         │
         │ gRPC (internal, fast)
         ├──► User Service
         ├──► Order Service
         ├──► Payment Service
         └──► Inventory Service
                │
                │ gRPC (internal)
                └──► Warehouse Service
```

**Rule of thumb:**

- External-facing → REST
- Internal service-to-service → gRPC

---

## 📡 gRPC Streaming Modes

| Mode             | Use Case                     | Example              |
| ---------------- | ---------------------------- | -------------------- |
| Unary            | Regular request-response     | Get user             |
| Server streaming | Server pushes many responses | Live dashboard       |
| Client streaming | Client uploads many chunks   | File upload          |
| Bi-directional   | Chat, real-time games        | WebSocket equivalent |

---

## ✅ Pros

**REST:** Universal, easy to debug, browser-native, no code gen  
**gRPC:** Fast, binary compact, streaming, type-safe auto-generated code

## ❌ Cons

**REST:** Verbose payload, no native streaming, no schema enforcement  
**gRPC:** Not browser-friendly, harder to debug, .proto schema required

## ⚖️ When to Use / When NOT to Use

**✅ gRPC — use when:**

- Internal microservice communication at high throughput
- Polyglot services (Go calling Python calling Java)
- Need bi-directional streaming (live data, game state)
- Schema enforcement and type safety across teams

**✅ REST — use when:**

- Public APIs for external developers
- Browser-based clients
- Simple CRUD with low traffic
- Rapid prototyping (no schema setup overhead)

**❌ Avoid gRPC when:**

- Browser clients must call directly (use REST + internal gRPC)
- Team unfamiliar with protobuf (overhead to learn)
- Simple services with minimal inter-service traffic

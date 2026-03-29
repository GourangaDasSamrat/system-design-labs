# 🖥️ Client-Server Model

> **One-liner:** A client requests resources/services, and a server responds with them. Every web interaction follows this model.

---

## 📌 What Is It?

The client-server model is the **foundational architecture** of the internet. It defines a relationship between two parties:

- **Client** — the requester (browser, mobile app, CLI tool)
- **Server** — the provider (web server, API server, database server)

This is a **request-response** cycle. The client always initiates; the server always reacts.

---

## 🔁 How a Request Works (Step by Step)

1. You type `google.com` in the browser
2. Browser checks its **local DNS cache**
3. If not found → asks **OS DNS cache**
4. If not found → asks **ISP's DNS resolver**
5. DNS resolver returns the **IP address** (e.g., `142.250.195.78`)
6. Browser opens a **TCP connection** to that IP on port 80/443
7. Browser sends an **HTTP request**
8. Server processes the request and sends back an **HTTP response**
9. Browser renders the HTML/CSS/JS

---

## 🧩 Key Components

| Component      | Role                                               |
| -------------- | -------------------------------------------------- |
| **Client**     | Initiates requests (browser, app)                  |
| **Server**     | Handles requests, returns responses                |
| **Protocol**   | Rules for communication (HTTP, WebSocket, gRPC)    |
| **IP Address** | Unique address of the server on the network        |
| **Port**       | Channel on the IP (80=HTTP, 443=HTTPS, 3306=MySQL) |

---

## 🌐 Types of Clients

- **Thin Client** — Only rendering/display logic (browser)
- **Thick Client** — Has business logic locally (desktop apps)
- **Mobile Client** — App that hits APIs

---

## 🔄 Communication Models

### Request-Response (HTTP)

```
Client ──── GET /users ────► Server
Client ◄─── 200 OK + data ── Server
```

### Long Polling

```
Client ──── GET /updates ──► Server (holds connection)
Server ◄─── responds ONLY when data is ready
```

### WebSockets (Bi-directional)

```
Client ◄──────────────────► Server
      (real-time, both ways)
```

---

## 🏗️ Single Server Architecture

```
[User's Browser]
      |
      | HTTP Request
      ▼
[Web + App Server] ──── reads/writes ────► [Database]
      |
      | HTTP Response
      ▼
[User's Browser]
```

**Problems with a single server:**

- Single Point of Failure (SPOF)
- Can't scale horizontally
- Web traffic + DB on same machine = resource contention

---

## 📊 Real-World Analogy

Think of it like a **restaurant**:

- **Client** = Customer placing an order
- **Server** = Waiter taking the order
- **Database** = Kitchen preparing the food
- **Protocol** = Language/menu used to communicate

---

## ⚠️ Limitations of Simple Client-Server

| Problem              | Impact                     |
| -------------------- | -------------------------- |
| Single server → SPOF | Entire system goes down    |
| No caching           | Every request hits DB      |
| Stateful server      | Hard to scale horizontally |
| Tight coupling       | Hard to change one part    |

---

## 🔑 Key Takeaways

- The client always **initiates** the connection
- HTTP is **stateless** — server doesn't remember previous requests
- Modern systems have **many layers** between client and actual data
- Every "server" is just another computer listening on a port

---

## 🔗 Related Topics

- [DNS](./dns.md)
- [HTTP & HTTPS](./http-https.md)
- [Load Balancers](../04-networking-and-routing/load-balancers.md)
- [Single Server → Scale](./single-server.md)

# 🖥️ Single Server Setup

> **One-liner:** The simplest possible architecture — one machine running everything. Great for starting out, but a single point of failure.

---

## 📌 What Is It?

```
[Browser] ─── HTTP ──► [Single Server]
                          ├── Web Server (Nginx/Apache)
                          ├── App Code (Node/Python/Java)
                          └── Database (MySQL/PostgreSQL)
```

One machine handles everything: web serving, business logic, database.

---

## ✅ When It's Fine

- Prototype / MVP stage
- <1,000 daily users
- Internal tool / side project
- Learning / development

---

## ❌ Problems

| Problem                 | Impact                        |
| ----------------------- | ----------------------------- |
| **SPOF**                | Server dies → everything down |
| **No scaling**          | Can't add more machines       |
| **Resource contention** | App and DB fight for CPU/RAM  |
| **Hard to update**      | Any deploy = full downtime    |

---

## 🚀 First Step: Separate App & DB

```
[Browser] → [App Server] → [DB Server]
```

Now you can scale each independently.

---

## 🔗 Related Topics

- [Vertical Scaling](../02-scaling/vertical-scaling.md)
- [Horizontal Scaling](../02-scaling/horizontal-scaling.md)
- [Back-of-Envelope](./back-of-envelope.md)

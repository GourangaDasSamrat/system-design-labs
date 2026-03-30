# 🌐 IP & Networking Basics

> **One-liner:** IP (Internet Protocol) is the addressing system of the internet — every device gets a unique address, and packets are routed between them.

---

## 📌 IP Addresses

### IPv4

- 32-bit address → 2³² = ~4.3 billion addresses (running out!)
- Format: `192.168.1.1` (four 8-bit octets, 0–255)
- Private ranges (not routable on internet):
  - `10.0.0.0/8`
  - `172.16.0.0/12`
  - `192.168.0.0/16`

### IPv6

- 128-bit address → 2¹²⁸ = ~340 undecillion addresses
- Format: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- Solves IPv4 exhaustion problem

---

## 🔢 Ports

A port is a **logical channel** on an IP address. One server can run many services on different ports.

| Port  | Service      |
| ----- | ------------ |
| 22    | SSH          |
| 25    | SMTP (email) |
| 53    | DNS          |
| 80    | HTTP         |
| 443   | HTTPS        |
| 3306  | MySQL        |
| 5432  | PostgreSQL   |
| 6379  | Redis        |
| 27017 | MongoDB      |

Ports 0–1023 = well-known (require root). Ports 1024–65535 = available.

---

## 📦 TCP vs UDP

### TCP (Transmission Control Protocol)

- Connection-oriented — 3-way handshake (SYN, SYN-ACK, ACK)
- **Reliable** — guarantees delivery, ordering, error checking
- **Slower** — overhead of acknowledgments
- Use: HTTP, HTTPS, SSH, databases

### UDP (User Datagram Protocol)

- Connectionless — just fire and forget
- **Unreliable** — no delivery guarantee, no ordering
- **Fast** — no handshake overhead
- Use: DNS, video streaming, gaming, VoIP

```
TCP: SYN ──► SYN-ACK ◄── ACK (connection established, then data)
UDP: DATA ──► (no response needed, no connection)
```

---

## 🏠 NAT (Network Address Translation)

Your router at home has ONE public IP. All your devices share it via NAT.

```
Device (192.168.1.5:1234) ──► Router ──► Internet (Public IP:5000)
                               NAT table: 192.168.1.5:1234 ↔ PublicIP:5000
```

This is why IPv4 hasn't completely run out — NAT multiplies addresses.

---

## 🔗 OSI Model (7 Layers)

| Layer | Name         | Example              |
| ----- | ------------ | -------------------- |
| 7     | Application  | HTTP, DNS, SSH       |
| 6     | Presentation | TLS, encryption      |
| 5     | Session      | Session management   |
| 4     | Transport    | TCP, UDP             |
| 3     | Network      | IP routing           |
| 2     | Data Link    | Ethernet, MAC        |
| 1     | Physical     | Cables, WiFi signals |

For system design: mostly care about L3 (IP), L4 (TCP/UDP), L7 (HTTP).

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/http-tls-handshake.excalidraw`](../diagrams/http-tls-handshake.excalidraw)

---

## 🔑 Key Takeaways

- Every server has an IP + port — that's how clients find it
- TCP = reliable but slower; UDP = fast but unreliable
- Load balancers can operate at L4 (TCP) or L7 (HTTP)

---

## 🔗 Related Topics

- [DNS](./dns.md)
- [HTTP & HTTPS](./http-https.md)
- [Load Balancers](../04-networking-and-routing/load-balancers.md)

# 🔄 Reverse Proxy

> **One-liner:** A reverse proxy sits in front of your servers, accepting client requests and forwarding them — hiding your actual server infrastructure.

---

## 📌 Forward Proxy vs Reverse Proxy

```
Forward Proxy: Client → [Proxy] → Internet
  (client uses proxy to reach internet — hides client)

Reverse Proxy: Client → [Reverse Proxy] → Servers
  (sits in front of servers — hides servers)
```

---

## 🛠️ What Reverse Proxies Do

- **Hide internal servers** — clients never see real IPs
- **SSL termination** — handle HTTPS, pass HTTP internally
- **Load balancing** — distribute across backend servers
- **Caching** — cache responses at proxy level
- **Compression** — gzip responses
- **Static file serving** — serve images/CSS/JS directly
- **Rate limiting** — basic traffic control

---

## 🆚 Reverse Proxy vs API Gateway vs Load Balancer

```
Load Balancer: Distribute traffic (L4 or L7)
Reverse Proxy: Hide servers + SSL + static files + basic routing
API Gateway:   All of above + auth + rate limiting + transformation + monitoring

(In practice: Nginx can do all three)
```

---

## 📝 Nginx as Reverse Proxy

```nginx
server {
    listen 443 ssl;
    server_name api.myapp.com;

    ssl_certificate /etc/ssl/cert.pem;
    ssl_certificate_key /etc/ssl/key.pem;

    # Reverse proxy to Node.js app
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Serve static files directly (no proxy)
    location /static/ {
        root /var/www;
        expires 1y;
    }
}
```

---

## 🔗 Related Topics

- [Load Balancers](./load-balancers.md)
- [API Gateway](./api-gateway.md)
- [CDN](./cdn.md)

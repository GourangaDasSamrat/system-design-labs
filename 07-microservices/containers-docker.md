# 🐳 Containers & Docker

> **One-liner:** Containers package your application and all its dependencies into one portable unit that runs identically on any machine — eliminating "works on my machine" problems.

---

## 📌 The Problem Before Containers

```
Dev machine:    Node 18, npm 9, Ubuntu 22
Staging server: Node 16, npm 8, CentOS 7
Production:     Node 14, npm 6, Amazon Linux 2

Result: App works in dev, breaks in prod 🔥
```

---

## 💡 Containers vs Virtual Machines

### Virtual Machine (VM)

```
[App A] [App B] [App C]
[Guest OS] [Guest OS] [Guest OS]   ← each VM has full OS (GBs)
[Hypervisor (VMware, KVM)]
[Host OS]
[Hardware]
```

Heavy: ~1 GB+ per VM, slow to start (minutes)

### Container

```
[App A] [App B] [App C]
[Container Runtime (Docker)]       ← share host OS kernel
[Host OS]
[Hardware]
```

Lightweight: ~10-100 MB per container, starts in milliseconds

---

## 🐳 Docker Core Concepts

### Image

A read-only template — like a blueprint/snapshot of your app + dependencies.

```dockerfile
# Dockerfile — recipe to build an image
FROM node:18-alpine          # base image
WORKDIR /app                 # working directory
COPY package*.json ./        # copy dependency files
RUN npm ci --production      # install dependencies
COPY . .                     # copy source code
EXPOSE 3000                  # document port
CMD ["node", "server.js"]    # start command
```

Build: `docker build -t my-app:1.0 .`

### Container

A running instance of an image.

```bash
docker run -d \
  -p 3000:3000 \
  -e DATABASE_URL=postgres://... \
  --name my-app \
  my-app:1.0
```

### Registry

Where images are stored and shared.

```bash
docker push myrepo/my-app:1.0   # push to Docker Hub / ECR
docker pull myrepo/my-app:1.0   # pull on another machine
```

Popular registries: Docker Hub, AWS ECR, GitHub Container Registry, GCR

---

## 📁 Dockerfile Best Practices

### Layer Caching — Order Matters!

```dockerfile
# ❌ Bad — any code change invalidates npm install layer
FROM node:18-alpine
COPY . .
RUN npm ci

# ✅ Good — npm install layer cached unless package.json changes
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./   ← copy just package files first
RUN npm ci              ← this layer cached if package.json unchanged
COPY . .                ← code changes only bust this layer
CMD ["node", "server.js"]
```

### Multi-Stage Build (Small Final Image)

```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

# Stage 2: Production (only compiled output)
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]
```

Result: Final image is tiny (no dev tools, no source, no test files).

### .dockerignore

```
node_modules
.git
*.test.js
.env
dist
```

---

## 🔧 Docker Compose — Multi-Container Local Dev

```yaml
# docker-compose.yml
version: "3.8"
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://postgres:pass@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

volumes:
  postgres_data:
```

Start everything: `docker compose up -d`  
Stop: `docker compose down`

---

## ☸️ Container Orchestration: Kubernetes (K8s)

Docker runs containers on ONE machine. In production you need many machines.

**Kubernetes** orchestrates containers across a cluster of machines.

### Key K8s Concepts

| Concept        | What It Is                                          |
| -------------- | --------------------------------------------------- |
| **Pod**        | Smallest unit — one or more containers              |
| **Deployment** | Manages pod replicas, rolling updates               |
| **Service**    | Stable network endpoint for a set of pods           |
| **Ingress**    | HTTP routing into the cluster (like an API Gateway) |
| **ConfigMap**  | Non-sensitive config (env vars)                     |
| **Secret**     | Sensitive config (passwords, API keys)              |
| **Namespace**  | Logical isolation within a cluster                  |

```yaml
# Deployment — run 3 replicas of my-app
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    spec:
      containers:
        - name: my-app
          image: myrepo/my-app:1.0
          ports:
            - containerPort: 3000
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
            limits:
              memory: "256Mi"
              cpu: "500m"
```

### K8s Benefits

- **Auto-scaling** — scale pods based on CPU/memory
- **Self-healing** — restart failed containers automatically
- **Rolling updates** — deploy new version with zero downtime
- **Load balancing** — distribute traffic across pods
- **Secret management** — inject secrets as env vars

### Managed K8s Services

| Cloud  | Service                          |
| ------ | -------------------------------- |
| AWS    | EKS (Elastic Kubernetes Service) |
| Google | GKE (Google Kubernetes Engine)   |
| Azure  | AKS (Azure Kubernetes Service)   |

---

## 🔄 Container in CI/CD Pipeline

```
Developer pushes code →
[GitHub Actions / Jenkins]
  1. Run tests
  2. Build Docker image: docker build -t app:$GIT_SHA .
  3. Push to ECR: docker push app:$GIT_SHA
  4. Deploy to K8s: kubectl set image deployment/app app=app:$GIT_SHA
  5. K8s rolling update → zero-downtime deploy
```

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/containers-docker.excalidraw`](../diagrams/containers-docker.excalidraw)

The diagram shows:

- VM vs Container architecture layers
- Docker image → container lifecycle
- Docker Compose multi-service local setup
- K8s cluster with nodes, pods, services

---

## 🔑 Key Takeaways

- Containers solve the **environment consistency** problem — build once, run anywhere
- Always use **multi-stage builds** to keep images small
- **Docker Compose** for local dev; **Kubernetes** for production orchestration
- Container images should be **immutable** — no SSH into containers to fix things; rebuild and redeploy

---

## 🔗 Related Topics

- [Microservices](./microservices.md)
- [Horizontal Scaling](../02-scaling/horizontal-scaling.md)
- [API Gateway](../04-networking-and-routing/api-gateway.md)

# System Design Notes

> A structured, self-paced knowledge base for mastering system design — from core fundamentals to production-grade architectural patterns.

---

## Overview

This repository contains personal study notes, diagrams, and reference material focused on building a deep, practical understanding of large-scale distributed systems. The content is organized to support both interview preparation and real-world engineering insight.

**Goals:**

- Build a strong foundation in distributed systems and architecture
- Understand trade-offs between competing design choices
- Prepare effectively for system design interviews at top-tier companies

---

## Visual Roadmap

Open **[roadmap.excalidraw](./rodemap.excalidraw)** with [excalidraw.com](https://excalidraw.com) or the VS Code Excalidraw extension to view the full interactive learning roadmap.

The roadmap is organized into **6 zones** (color-coded by difficulty) with flow arrows, pattern cheat sheets, and diagram links for every topic.

| Color     | Difficulty             |
| --------- | ---------------------- |
| 🟢 Green  | Beginner / Basics      |
| 🟡 Yellow | Intermediate           |
| 🔴 Red    | Advanced               |
| 🟣 Purple | Pattern Hub / Diagrams |

---

## Repository Structure

```
system-design-notes/
├── 01-fundamentals/                  # Client-Server, DNS, IP, HTTP
├── 02-scaling/                       # Vertical, Horizontal, Consistent Hashing
├── 03-databases/                     # SQL, NoSQL, Replication, Sharding
├── 04-networking-and-routing/        # Load Balancer, API Gateway, Reverse Proxy, CDN
├── 05-caching/                       # Caching strategies, Redis
├── 06-async-communication/           # Queues (SQS), Pub-Sub (SNS), Fan-out
├── 07-microservices/                 # Microservices, Containers, gRPC, Saga, Versioning
├── 08-reliability-and-performance/   # Rate Limiting, Bloom Filters, CAP Theorem
├── 09-hld-case-studies/              # URL Shortener, Twitter, Instagram, Notification System
├── 10-interview-prep/                # Framework, common Qs, cheat sheets
├── 11-security/                      # Auth (JWT/OAuth), DDoS Protection
├── 12-modern-patterns/               # Distributed Locks, Idempotency, Circuit Breaker, Observability
├── 13-multi-region/                  # Geo-distribution, Replication, DR, RTO/RPO
├── 14-cost-optimization/             # Storage Tiering, Caching vs Compute, Spot Instances
├── 15-edge-computing/                # Edge Functions, Cloudflare Workers, Use Cases
├── 16-failure-resilience/            # Retry/Backoff, DLQ, Graceful Degradation, Chaos Eng.
├── 17-api-design-advanced/           # Pagination, Filtering, GraphQL Basics, Versioning
├── patterns/                         # Design Patterns: Leader Election, Cron, Rate Limiters, Cache
├── diagrams/                         # Excalidraw diagrams (.excalidraw) for every topic
└── roadmap.excalidraw                # Visual learning roadmap (open in excalidraw.com)
```

---

## Opening Diagrams

Every topic includes a corresponding `.excalidraw` diagram located in the `diagrams/` folder.

**Option 1 — Browser:**

1. Go to [excalidraw.com](https://excalidraw.com)
2. Click **≡** (top-left menu) → **Open**
3. Select the relevant `.excalidraw` file from the `diagrams/` folder

**Option 2 — VS Code:**
Install the [Excalidraw extension](https://marketplace.visualstudio.com/items?itemName=pomdtr.excalidraw-editor) and open any `.excalidraw` file directly from the editor.

---

## Interview Framework

A repeatable, structured approach to tackling any system design question:

```
1. Clarify Requirements     →  Functional & non-functional (scale, latency, availability)
2. Estimate Scale           →  DAU, QPS, storage needs, bandwidth
3. API Design               →  Endpoints, methods, request/response contracts
4. Database Schema          →  Tables, indexes, shard key selection
5. High-Level Design        →  Core components and data flow
6. Deep Dive                →  Detail 2–3 critical components
7. Scaling Strategy         →  Identify bottlenecks and mitigation approaches
8. Trade-offs               →  Justify your choices; acknowledge alternatives
```

> Full framework with templates → [`10-interview-prep/framework.md`](10-interview-prep/framework.md)

---

## The Trade-offs Mindset

Most topics in this repository is analyzed through the following lens:

| Dimension           | Description                             |
| ------------------- | --------------------------------------- |
| **Pros**            | What this approach does well            |
| **Cons**            | Its limitations and failure modes       |
| **When to use**     | The right context for this choice       |
| **When NOT to use** | Where it breaks down or introduces risk |

This structure directly mirrors what interviewers are looking for when they ask _"Why did you choose X over Y?"_

---

## Resources

| Tool                                                                                                         | Purpose                                       |
| ------------------------------------------------------------------------------------------------------------ | --------------------------------------------- |
| [Excalidraw](https://excalidraw.com)                                                                         | Open and view all `.excalidraw` diagram files |
| [Excalidraw VS Code Extension](https://marketplace.visualstudio.com/items?itemName=pomdtr.excalidraw-editor) | Edit diagrams directly in your editor         |

---

## License

Copyright © 2026 **Gouranga Das Samrat**. All rights reserved.

This work is licensed under the **[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/)** (CC BY-NC-SA 4.0).

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

**You are free to:**
- ✅ **Share** — copy and redistribute the material in any medium or format
- ✅ **Adapt** — remix, transform, and build upon the material

**Under the following terms:**
- 📛 **Attribution** — You must give appropriate credit to [Gouranga Das Samrat](https://github.com/GoruangaDasSamrat), provide a link to the license, and indicate if changes were made.
- 🚫 **NonCommercial** — You may not use the material for commercial purposes.
- 🔄 **ShareAlike** — If you remix or build upon this material, you must distribute your contributions under the same license.

> Unauthorized commercial use or redistribution without proper attribution is strictly prohibited.

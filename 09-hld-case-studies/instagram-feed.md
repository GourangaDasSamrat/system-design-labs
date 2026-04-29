# 📸 HLD: Instagram Feed

> Design a photo-sharing platform where users post photos and see a feed of photos from people they follow.

---

## 1️⃣ Clarify Requirements

### Functional Requirements

- Upload photos/videos
- Follow/unfollow users
- View home feed (photos from followed users, newest first)
- Like, comment on photos
- User profile with photo grid

### Non-Functional Requirements

- High availability
- Eventual consistency for feed (seconds lag is fine)
- Low latency feed load < 200ms
- **Image-heavy** — media storage is the core challenge
- Scale: 500M DAU, 100M photos uploaded/day

---

## 2️⃣ Estimate Scale

```
Photo uploads/day: 100M
Write QPS: 100M / 86,400 ≈ 1,160 uploads/sec
Read QPS: 500M × 10 feed loads/day / 86,400 ≈ 58,000 reads/sec

Storage:
  Avg photo: 3 MB (original) + thumbnails (100KB, 500KB) ≈ 4 MB total
  100M photos/day × 4 MB = 400 TB/day
  5-year storage: ~730 PB (need tiered storage!)

CDN bandwidth:
  58K reads/sec × 10 photos/feed × 500KB avg = ~290 GB/sec outbound
```

---

## 3️⃣ Architecture Overview

```
[Client]
   │
[CDN] ← photos served here (not from origin)
   │
[API Gateway + LB]
   │
   ├── [Photo Service] → upload → [S3] → trigger → [Resize Worker]
   │                                                      → S3 (thumbnail, medium, large)
   │                                                      → Update metadata DB
   │
   ├── [Feed Service] → Redis (pre-computed feed) → return photoIds → CDN URLs
   │
   ├── [User Service] → PostgreSQL
   │
   └── [Social Graph] → Cassandra (followers/following)
```

---

## 4️⃣ Photo Upload Flow

```
1. Client → POST /upload → Photo Service
2. Photo Service → generate uploadId → return S3 pre-signed URL
3. Client → upload directly to S3 (bypasses your servers!)
4. S3 → triggers Lambda / sends event to queue
5. Resize Worker → create 3 sizes → store in S3
6. Update metadata DB: photoId, userId, S3 keys, created_at
7. Fan-out → push photoId to followers' feed caches (async)
```

**Pre-signed URL pattern** = your servers never touch the image bytes.

---

## 5️⃣ Feed Generation (Hybrid Fan-out)

Same pattern as Twitter:

- Regular users → fan-out on write
- Celebrities → fan-out on read, merge at read time

```
Redis feed cache:
  Key: feed:{userId}
  Value: Sorted Set of {photoId, timestamp}
  Size: 500 entries max
  TTL: 7 days
```

---

## 6️⃣ Database Schema

```sql
-- Photos
CREATE TABLE photos (
  photo_id    BIGINT PRIMARY KEY,  -- Snowflake ID
  user_id     BIGINT NOT NULL,
  s3_key_orig VARCHAR(500),
  s3_key_med  VARCHAR(500),
  s3_key_thumb VARCHAR(500),
  caption     TEXT,
  created_at  TIMESTAMP DEFAULT NOW(),
  likes_count BIGINT DEFAULT 0
);

-- Social graph (Cassandra)
CREATE TABLE following (
  follower_id BIGINT,
  followed_id BIGINT,
  PRIMARY KEY (follower_id, followed_id)
);
```

---

## 7️⃣ Key Challenges

### Storage Tiering

```
Hot (0-7 days):   S3 Standard (fast, expensive)
Warm (7-90 days): S3 Infrequent Access (cheaper)
Cold (90+ days):  S3 Glacier (very cheap, slow retrieval)
```

### Serving Photos

- NEVER serve from origin → always from CDN
- URL format: `https://cdn.instagram.com/photos/{photoId}/{size}.jpg`
- Immutable URLs (content never changes for same ID) → cache forever

---

## ✅ Trade-offs Summary

| Decision       | Choice         | Rationale                                  |
| -------------- | -------------- | ------------------------------------------ |
| Photo storage  | S3 + CDN       | Scalable object store, edge serving        |
| Upload pattern | Pre-signed URL | Don't bottleneck servers with bytes        |
| Feed           | Hybrid fan-out | Celebrities break pure push                |
| Social graph   | Cassandra      | Write-heavy, large-scale relationships     |
| Image sizes    | 3 variants     | Different contexts (thumbnail, feed, full) |

---

## 🔗 Related Topics

- [CDN](../04-networking-and-routing/cdn.md)
- [Fan-out Architecture](../06-async-communication/fan-out.md)
- [HLD: Twitter](./twitter.md)
- [Message Queues](../06-async-communication/message-queues-sqs.md)

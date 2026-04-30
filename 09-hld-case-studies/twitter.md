# 🐦 HLD: Twitter / X

> Design a social media platform where users post tweets, follow each other, and see a timeline of tweets from accounts they follow.

---

## 1️⃣ Clarify Requirements

### Functional Requirements

- Post a tweet (text, images, videos)
- Follow / unfollow users
- View home timeline (tweets from followed users, newest first)
- Like, retweet, reply
- User profile with tweet history
- Search tweets

### Non-Functional Requirements

- **High availability** — Twitter is a utility
- **Eventual consistency** — timeline can be slightly stale (seconds)
- **Low latency** — timeline load < 200ms
- **Read-heavy** — 100:1 read-to-write ratio
- Scale: 300M DAU, 500M tweets/day

---

## 2️⃣ Estimate Scale

```
Tweets/day: 500M
Write QPS:  500M / 86,400 ≈ 5,800 writes/sec
Read QPS:   5,800 × 100 = 580,000 reads/sec
Peak read:  ~1-2M reads/sec

Storage:
  Tweet: 280 chars = ~300 bytes + metadata ≈ 1 KB
  500M tweets/day × 1 KB = 500 GB/day
  Media: ~100 TB/day (images, videos)

Timeline cache:
  300M users × 100 tweet IDs × 8 bytes = 240 GB
```

---

## 3️⃣ API Design

```
POST /tweets
Body: { text, mediaIds[] }
Response: { tweetId, createdAt }

GET /timeline/home?cursor=&limit=20
Response: { tweets: [...], nextCursor }

GET /users/:userId/tweets?cursor=
Response: { tweets: [...], nextCursor }

POST /tweets/:tweetId/likes
DELETE /tweets/:tweetId/likes

POST /users/:userId/follow
DELETE /users/:userId/follow
```

---

## 4️⃣ The Core Problem: Home Timeline

The hardest part. User A follows 500 people. How do you fetch their timeline?

### Approach 1: Pull (Fan-out on Read)

```
Timeline request:
1. Get all 500 followed user IDs
2. Query tweets from each user (last 24h)
3. Merge and sort by time
4. Return top 20
```

❌ Slow: 500 DB queries per timeline request  
❌ Gets worse as you follow more people  
❌ Doesn't scale at 580K reads/sec

### Approach 2: Push (Fan-out on Write) — Twitter's Original Approach

```
User A tweets:
1. Find all A's followers (e.g., 1,000 followers)
2. Push tweet ID to each follower's timeline cache
3. Timeline request → just read from your cached timeline
```

```
Tweet created → [Fan-out Service]
                    → Redis: user1_timeline.lpush(tweetId)
                    → Redis: user2_timeline.lpush(tweetId)
                    → Redis: user3_timeline.lpush(tweetId)
                    ... (1,000 followers)
```

✅ Timeline reads are O(1) — just read from Redis  
❌ Celebrity problem: Lady Gaga has 100M followers → 100M Redis writes per tweet  
❌ Wasted if followers don't check timeline

### Approach 3: Hybrid (Twitter's Current Approach)

```
Regular users (< 1M followers): Fan-out on Write
Celebrity users (> threshold):  Fan-out on Read

Timeline request:
1. Fetch pre-computed timeline from Redis (fan-out writes)
2. Fetch recent tweets from celebrity accounts you follow (fan-out reads)
3. Merge both, sort, return
```

---

## 5️⃣ High-Level Architecture

```
[Client]
   │
   ▼
[CDN] ← static assets, media
   │
[API Gateway + LB]
   │
   ├──────────────────────────────────────┐
   │                                      │
[Tweet Service]                   [Timeline Service]
   │                                      │
   ├─► [Media Service] → [S3/CDN]         ├─► [Timeline Cache (Redis)]
   │                                      │         (500 tweet IDs per user)
   └─► [Tweets DB]                        └─► [Fan-out Service]
       (Cassandra: write-heavy,                   │
        time-series)                        [Message Queue (Kafka)]
                                                  │
                                        [Worker Pool: fan-out writers]
                                                  │
                                          [Timeline Cache Redis]
```

---

## 6️⃣ Database Design

### Tweets Table (Cassandra)

```sql
CREATE TABLE tweets (
  tweet_id     BIGINT,          -- Snowflake ID (time-sortable)
  user_id      BIGINT,
  text         TEXT,
  media_ids    LIST<UUID>,
  created_at   TIMESTAMP,
  likes_count  COUNTER,
  retweet_count COUNTER,
  PRIMARY KEY (user_id, tweet_id)  -- partition by user, sort by tweet
) WITH CLUSTERING ORDER BY (tweet_id DESC);
```

Why Cassandra?

- Write-heavy (500M tweets/day)
- Time-series access pattern (recent tweets)
- Naturally partitioned by user_id

### Followers Table (Graph in Cassandra)

```sql
-- Who does user X follow?
CREATE TABLE following (
  follower_id  BIGINT,
  followed_id  BIGINT,
  created_at   TIMESTAMP,
  PRIMARY KEY (follower_id, followed_id)
);

-- Who follows user X?
CREATE TABLE followers (
  followed_id  BIGINT,
  follower_id  BIGINT,
  created_at   TIMESTAMP,
  PRIMARY KEY (followed_id, follower_id)
);
```

Denormalized for O(1) lookup in both directions.

### Timeline Cache (Redis)

```
Key: timeline:{userId}
Value: Sorted Set of tweet IDs (score = timestamp)
Size: Keep last 800 tweet IDs
TTL: 7 days (trim inactive users)
```

---

## 7️⃣ Media Storage

```
Tweet with image:
1. Client uploads image → [Media Service]
2. Media Service → S3 (original)
3. Media Service → async resize: thumbnail, medium, large
4. All sizes stored in S3
5. CloudFront CDN in front of S3
6. Tweet stores mediaId → resolved to CDN URL on read
```

---

## 8️⃣ Search

```
Tweets → Elasticsearch
Index: tweet_id, text, user_id, created_at, hashtags

Search query → Elasticsearch → ranked results
Trending hashtags → pre-computed every 5 min, stored in Redis
```

---

## 9️⃣ The Celebrity (Hotspot) Problem

```
Elon Musk tweets → 100M followers
Fan-out on write: 100M Redis writes in seconds → IMPOSSIBLE

Solution:
- Maintain a "high-follower" list
- Skip fan-out for celebrities
- At read time: fetch celebrity tweets separately, merge with pre-computed timeline
- Cache celebrity recent tweets (they're read by millions)
```

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/hld-twitter.excalidraw`](../diagrams/hld-twitter.excalidraw)

The diagram shows:

- Full architecture: Client → CDN → Gateway → Services
- Fan-out on write flow (Kafka → workers → Redis)
- Hybrid timeline assembly (pre-computed + celebrity merge)
- Cassandra tweet storage partition scheme
- Media upload and CDN serving path

---

## ✅ Trade-offs Summary

| Decision            | Choice           | Rationale                         |
| ------------------- | ---------------- | --------------------------------- |
| Timeline generation | Hybrid push/pull | Pure push fails for celebrities   |
| Tweet storage       | Cassandra        | Write-heavy, time-series          |
| Timeline cache      | Redis Sorted Set | O(log N) insert, O(1) range read  |
| Fan-out             | Async via Kafka  | Don't block tweet creation        |
| Media               | S3 + CDN         | Scalable object storage           |
| Consistency         | Eventual         | Timeline can be 1-2s stale — fine |

---

## 🔗 Related Topics

- [Fan-out Architecture](../06-async-communication/pub-sub-sns.md)
- [Redis](../05-caching/redis.md)
- [Consistent Hashing](../02-scaling/consistent-hashing.md)
- [CDN](../04-networking-and-routing/cdn.md)

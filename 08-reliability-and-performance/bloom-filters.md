# 🌸 Bloom Filters

> **One-liner:** A probabilistic data structure that tells you if an element is **definitely not** in a set, or **possibly** in a set — using very little memory.

---

## 📌 The Problem

You have 1 billion URLs in a database. Before adding a new URL, you want to check if it already exists.

**Naive approach:** Query the database every time.

- Cost: 1 DB query per URL check → slow, expensive

**Bloom Filter approach:** Check the bloom filter first (microseconds, no DB hit).

- If filter says NO → URL definitely not in DB → safe to insert
- If filter says YES → URL might be in DB → query DB to confirm

---

## 💡 How Bloom Filters Work

### Structure

An array of `m` bits, all initialized to 0.

```
Bit array: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]  (m=10 bits)
            0  1  2  3  4  5  6  7  8  9
```

### Insertion (k hash functions)

To insert element `x`:

1. Hash `x` with `k` different hash functions
2. Each hash function returns a position in the bit array
3. Set those positions to 1

```
Insert "google.com":
  hash1("google.com") = 2 → bit[2] = 1
  hash2("google.com") = 5 → bit[5] = 1
  hash3("google.com") = 8 → bit[8] = 1

Bit array: [0, 0, 1, 0, 0, 1, 0, 0, 1, 0]
```

### Lookup

To check if `y` is in the set:

1. Hash `y` with the same `k` functions
2. Check if ALL those positions are 1

```
Check "google.com":
  hash1 → bit[2] = 1 ✅
  hash2 → bit[5] = 1 ✅
  hash3 → bit[8] = 1 ✅
  → POSSIBLY in set (correct! it was inserted)

Check "bing.com":
  hash1 → bit[1] = 0 ❌
  → DEFINITELY NOT in set (100% accurate)
```

---

## ⚠️ False Positives (Not False Negatives)

### False Positive Example

```
Insert "amazon.com":
  hash1 = 2, hash2 = 7, hash3 = 3

Bit array (after inserting google.com + amazon.com):
[0, 0, 1, 1, 0, 1, 0, 1, 1, 0]

Check "yahoo.com":
  hash1("yahoo.com") = 2 → bit[2] = 1 ✅
  hash2("yahoo.com") = 7 → bit[7] = 1 ✅
  hash3("yahoo.com") = 3 → bit[3] = 1 ✅
  → "POSSIBLY in set" ← FALSE POSITIVE! (yahoo.com was never inserted)
```

Bits were set by OTHER elements, creating a false positive.

### Key Guarantees

| Result                    | Guarantee                                    |
| ------------------------- | -------------------------------------------- |
| **DEFINITELY NOT in set** | 100% accurate (no false negatives)           |
| **POSSIBLY in set**       | Might be wrong (false positives possible)    |
| **Cannot delete**         | Once a bit is set, you can't safely unset it |

---

## 📊 Tuning Bloom Filters

Two parameters control accuracy:

- **m** = size of bit array (more bits → fewer false positives)
- **k** = number of hash functions (optimal k depends on m and n)
- **n** = number of expected elements

**False positive rate formula:**

```
p ≈ (1 - e^(-kn/m))^k
```

**Practical sizing:**

```
1% false positive rate → ~10 bits per element
0.1% false positive rate → ~15 bits per element

For 1 billion URLs, 1% FP rate:
→ 10 × 1B bits = 10 billion bits = 1.25 GB
vs
→ Storing 1B URLs as strings = ~50-100 GB
→ 40-80× memory savings!
```

---

## 🌍 Real-World Use Cases

| System                 | Use Case                                 |
| ---------------------- | ---------------------------------------- |
| **Google Chrome**      | Malicious URL check (local bloom filter) |
| **Apache Cassandra**   | Skip SSTables that don't contain a key   |
| **Bitcoin**            | SPV wallets filter transactions          |
| **Akamai CDN**         | Avoid caching one-hit-wonder URLs        |
| **Medium**             | "Have you seen this article?" check      |
| **Email spam filters** | Fast first-pass spam detection           |
| **HBase/BigTable**     | Reduce disk reads for non-existent rows  |

### Example: Web Crawler (Avoid Revisiting URLs)

```python
from pybloom_live import BloomFilter

bf = BloomFilter(capacity=1_000_000_000, error_rate=0.01)

def should_crawl(url):
    if url in bf:
        return False  # probably already crawled (or false positive)
    bf.add(url)
    return True  # definitely not crawled yet
```

### Example: Username Availability

```
User types username "rahul123"
→ Check bloom filter (microseconds)
→ Filter says NO → username definitely available ✅ (no DB query needed)
→ Filter says YES → query DB to confirm (might be false positive)
```

---

## 🆚 Bloom Filter vs Hash Set

| Feature         | Bloom Filter      | Hash Set            |
| --------------- | ----------------- | ------------------- |
| Memory          | Very small (bits) | Large (full values) |
| Lookup          | O(k) very fast    | O(1) fast           |
| False positives | Possible          | Never               |
| False negatives | Never             | Never               |
| Deletion        | Not supported     | Supported           |
| Count elements  | No                | Yes                 |

---

## 🔄 Variants

| Variant                   | Feature                                           |
| ------------------------- | ------------------------------------------------- |
| **Counting Bloom Filter** | Supports deletions (use counters instead of bits) |
| **Scalable Bloom Filter** | Grows dynamically as elements are added           |
| **Cuckoo Filter**         | Supports deletion, similar performance            |

---

## 🎨 Excalidraw Diagram

> Open this file in [Excalidraw](https://excalidraw.com): [`../diagrams/bloom-filter.excalidraw`](../diagrams/bloom-filter.excalidraw)

The diagram shows:

- Bit array with positions labeled
- Three hash functions pointing to different positions
- Insertion of "google.com" setting 3 bits
- Lookup showing definite miss vs possible hit
- False positive scenario with two elements colliding

---

## 🔑 Key Takeaways

- Bloom filters use **tiny memory** to check set membership
- **No false negatives** — if it says NO, it's definitely NO
- **False positives possible** — if it says YES, verify with DB
- Perfect for **pre-filtering** expensive DB/disk lookups
- Tune with bits-per-element to control false positive rate

---

## 🔗 Related Topics

- [Caching Strategies](../05-caching/caching-strategies.md)
- [Database Scaling](../03-databases/database-scaling.md)
- [Consistent Hashing](../02-scaling/consistent-hashing.md)

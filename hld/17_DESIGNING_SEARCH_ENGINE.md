# Designing a Search Engine (Google)

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Requirements Gathering](#requirements-gathering)
3. [Capacity Estimation](#capacity-estimation)
4. [High-Level Design](#high-level-design)
5. [Web Crawling](#web-crawling)
6. [Indexing](#indexing)
7. [Ranking](#ranking)

---

## Problem Statement

Design a search engine like Google that:
- Crawls the web
- Indexes web pages
- Ranks search results
- Returns results in < 100ms

---

## Requirements Gathering

### Functional Requirements

1. **Web Crawling**
   - Discover web pages
   - Download content
   - Follow links

2. **Indexing**
   - Extract keywords
   - Build inverted index
   - Store metadata

3. **Search**
   - Process queries
   - Rank results
   - Return top results

### Non-Functional Requirements

1. **Scale**
   - 1 trillion web pages
   - 5 billion searches/day
   - 100,000 QPS

2. **Performance**
   - Search: < 100ms
   - Index update: Real-time

---

## Capacity Estimation

### Traffic Estimates

**Searches:**
```
5B searches/day
= 5B / 86,400 seconds
= 57,870 QPS
Peak: 57,870 × 3 = 173,610 QPS ≈ 175K QPS
```

**Storage:**
```
1T web pages
Average: 50 KB per page
Total: 1T × 50 KB = 50 PB
With indexes: 50 PB × 2 = 100 PB
```

---

## High-Level Design

### Architecture

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     ▼
┌──────────────┐
│ Search       │
│ Service      │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│Index│ │Cache│
└─────┘ └─────┘
       │
       ▼
┌──────────────┐
│   Crawler    │
│   Service    │
└──────────────┘
```

### Components

1. **Crawler:** Discover and download web pages
2. **Indexer:** Build search index
3. **Search Service:** Process queries
4. **Ranking Service:** Rank results

---

## Web Crawling

### Crawling Process

```
1. Seed URLs (starting points)
2. Download pages
3. Extract links
4. Add new URLs to queue
5. Repeat
```

### Crawler Architecture

```
┌──────────────┐
│ URL Queue    │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   Crawler    │
│   Workers    │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│Store│ │Extract│
│Pages│ │Links │
└─────┘ └─────┘
```

### Challenges

1. **Robots.txt:** Respect website rules
2. **Rate Limiting:** Don't overwhelm servers
3. **Duplicate Content:** Detect and skip
4. **Dynamic Content:** Handle JavaScript

---

## Indexing

### Inverted Index

**Structure:**
```
Word → [Document IDs]

"cat" → [1, 5, 10, 25]
"dog" → [2, 5, 15]
"pet" → [1, 2, 5, 10]
```

**Query: "cat AND dog"**
```
"cat" → [1, 5, 10, 25]
"dog" → [2, 5, 15]
Intersection → [5]
```

### Index Building

```
1. Extract text from pages
2. Tokenize (split into words)
3. Remove stop words ("the", "a", etc.)
4. Stem words ("running" → "run")
5. Build inverted index
```

---

## Ranking

### PageRank Algorithm

**Basic Idea:**
- Pages linked by many pages are important
- Links from important pages count more

**Formula:**
```
PR(A) = (1-d) + d × Σ(PR(T)/C(T))
```

Where:
- PR(A): PageRank of page A
- d: Damping factor (0.85)
- T: Pages linking to A
- C(T): Outgoing links from T

### Ranking Factors

1. **PageRank:** Link-based importance
2. **TF-IDF:** Term frequency
3. **Content Quality:** Freshness, relevance
4. **User Signals:** Click-through rate

---

## Database Design

### Schema

```sql
CREATE TABLE pages (
    url VARCHAR(2048) PRIMARY KEY,
    title VARCHAR(255),
    content TEXT,
    crawled_at TIMESTAMP,
    pagerank DECIMAL(10,8)
);

CREATE TABLE inverted_index (
    word VARCHAR(100),
    page_url VARCHAR(2048),
    tf_idf DECIMAL(10,8),
    PRIMARY KEY (word, page_url),
    INDEX idx_word (word)
);
```

---

## Scaling the System

### 1. Distributed Crawling

```
Multiple crawlers:
- Geographic distribution
- Parallel crawling
- Load balancing
```

### 2. Index Sharding

```
Shard by word:
hash(word) % number_of_shards = shard_number
```

### 3. Caching

```
Cache popular queries:
- Top 1M queries cached
- 80% cache hit rate
- Reduce index load
```

---

## Summary

**Key Design Decisions:**
1. ✅ **Crawler:** Discover and download pages
2. ✅ **Inverted Index:** Fast keyword lookup
3. ✅ **PageRank:** Link-based ranking
4. ✅ **Sharding:** Scale index
5. ✅ **Caching:** Popular queries

**Next Steps:**
- Learn [Designing Payment System](./18_DESIGNING_PAYMENT_SYSTEM.md)
- Understand [Distributed Systems](./21_DISTRIBUTED_SYSTEMS.md)
- Practice [More System Designs](./README.md)

---

**References:**
- [Google Architecture](https://www.google.com)
- [Search Engine Design](https://algomaster.io/learn/system-design)

# Designing a URL Shortener (bit.ly)

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Requirements Gathering](#requirements-gathering)
3. [Capacity Estimation](#capacity-estimation)
4. [High-Level Design](#high-level-design)
5. [Detailed Design](#detailed-design)
6. [Database Design](#database-design)
7. [API Design](#api-design)
8. [Scaling the System](#scaling-the-system)
9. [Trade-offs and Optimizations](#trade-offs-and-optimizations)

---

## Problem Statement

Design a URL shortener service like bit.ly that:
- Converts long URLs to short URLs
- Redirects short URLs to original URLs
- Tracks analytics (click counts, timestamps)

**Example:**
```
Long URL:  https://www.example.com/very/long/path/to/resource?param=value
Short URL: https://short.ly/abc1234
```

---

## Requirements Gathering

### Functional Requirements

1. **URL Shortening**
   - Given a long URL, generate a short URL
   - Short URL should be unique
   - Short URL should be as short as possible (6-7 characters)

2. **URL Redirection**
   - Given a short URL, redirect to original URL
   - Handle expired URLs
   - Handle invalid URLs

3. **Analytics**
   - Track click counts
   - Track timestamps
   - Track geographic data (optional)

### Non-Functional Requirements

1. **Scale**
   - 100 million URLs shortened per day
   - 10 billion redirects per day (100:1 read/write ratio)
   - URLs stored for 2 years

2. **Performance**
   - URL shortening: < 200ms
   - URL redirection: < 100ms

3. **Availability**
   - 99.9% uptime
   - Handle failures gracefully

4. **Security**
   - Prevent abuse (rate limiting)
   - Prevent malicious URLs (optional)

---

## Capacity Estimation

### Traffic Estimates

**Write Operations (URL Shortening):**
```
100M URLs/day
= 100M / (24 × 60 × 60) seconds
= 1,160 URLs/second
≈ 1,200 URLs/second (with peak traffic)
```

**Read Operations (URL Redirection):**
```
100:1 read/write ratio
= 100M × 100 = 10B redirects/day
= 10B / (24 × 60 × 60) seconds
= 115,740 redirects/second
≈ 120,000 redirects/second (with peak traffic)
```

### Storage Estimates

**Per URL:**
```
Short URL:     7 bytes (e.g., "abc1234")
Long URL:      500 bytes (average)
Created At:    7 bytes (timestamp)
Expires At:    7 bytes (timestamp)
Click Count:   4 bytes (integer)
Total:        525 bytes per URL
```

**Total Storage:**
```
100M URLs/day × 365 days × 2 years
= 73 billion URLs

73B × 525 bytes
= 38.3 TB
≈ 40 TB (with indexes and overhead)
```

### Bandwidth Estimates

**Write Traffic:**
```
1,200 URLs/second × 500 bytes (long URL)
= 600 KB/second
= 4.8 Mbps
```

**Read Traffic:**
```
120,000 redirects/second × 500 bytes (response)
= 60 MB/second
= 480 Mbps
```

### Memory Estimates (Caching)

**Cache 20% of URLs (80/20 rule):**
```
20% of 73B URLs = 14.6B URLs
14.6B × 525 bytes = 7.7 TB

But we only cache hot URLs:
Top 20% of daily traffic = 20M URLs/day
20M × 525 bytes = 10.5 GB/day
Cache for 7 days = 73.5 GB
≈ 100 GB cache
```

---

## High-Level Design

### Basic Architecture

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     ▼
┌──────────────┐
│ Load         │
│ Balancer    │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│ API │ │ API │
│Srv 1│ │Srv 2│
└──┬──┘ └──┬──┘
   │       │
   └───┬───┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│Cache│ │ DB  │
│Redis│ │MySQL│
└─────┘ └─────┘
```

### Components

1. **API Servers:** Handle HTTP requests
2. **Load Balancer:** Distribute traffic
3. **Database:** Store URL mappings
4. **Cache:** Store hot URLs
5. **URL Generator:** Generate short URLs

---

## Detailed Design

### 1. URL Shortening Flow

```
┌─────────────────────────────────────┐
│ 1. Client sends long URL            │
│    POST /api/v1/shorten             │
│    { "long_url": "https://..." }    │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 2. API Server validates URL         │
│    - Check if valid URL format      │
│    - Check if already exists        │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 3. Generate short URL               │
│    - Hash long URL                  │
│    - Encode to base62               │
│    - Check for collisions           │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 4. Store in database                │
│    - Save mapping                   │
│    - Set expiration                 │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 5. Cache the mapping                │
│    - Store in Redis                 │
│    - Set TTL                        │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 6. Return short URL                │
│    { "short_url": "abc1234" }    │
└───────────────────────────────────┘
```

### 2. URL Redirection Flow

```
┌─────────────────────────────────────┐
│ 1. Client requests short URL        │
│    GET /abc1234                     │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 2. Check cache first                │
│    Redis: GET "abc1234"             │
└──────────────┬──────────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
   ┌────────┐   ┌────────┐
   │ Cache  │   │ Cache  │
   │  Hit   │   │  Miss  │
   └───┬────┘   └───┬────┘
       │             │
       │             ▼
       │      ┌──────────────┐
       │      │ 3. Query DB  │
       │      │ SELECT * FROM│
       │      │ urls WHERE   │
       │      │ short_url=...│
       │      └──────┬───────┘
       │             │
       │             ▼
       │      ┌──────────────┐
       │      │ 4. Update    │
       │      │ click_count  │
       │      └──────┬───────┘
       │             │
       │             ▼
       │      ┌──────────────┐
       │      │ 5. Cache it  │
       │      │ Redis SET    │
       │      └──────┬───────┘
       │             │
       └─────┬───────┘
             │
             ▼
┌─────────────────────────────────────┐
│ 6. Return 302 Redirect              │
│    Location: https://long-url...    │
└─────────────────────────────────────┘
```

---

## Database Design

### Schema

```sql
CREATE TABLE urls (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    short_url VARCHAR(7) UNIQUE NOT NULL,
    long_url VARCHAR(2048) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP,
    click_count BIGINT DEFAULT 0,
    INDEX idx_short_url (short_url),
    INDEX idx_expires_at (expires_at)
);

CREATE TABLE analytics (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    short_url VARCHAR(7) NOT NULL,
    clicked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address VARCHAR(45),
    user_agent VARCHAR(255),
    country VARCHAR(2),
    FOREIGN KEY (short_url) REFERENCES urls(short_url),
    INDEX idx_short_url (short_url),
    INDEX idx_clicked_at (clicked_at)
);
```

### Database Sharding

**Strategy: Hash-based sharding**

```
Shard Number = hash(short_url) % number_of_shards

Example with 10 shards:
"abc1234" → hash("abc1234") % 10 = 3 → Shard 3
"xyz5678" → hash("xyz5678") % 10 = 7 → Shard 7
```

**Architecture:**
```
┌─────────────────────────────────┐
│      Application Layer          │
└──────────────┬──────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
┌──────────┐   ┌──────────┐
│ Shard 1  │   │ Shard 2  │
│ (0-9%)   │   │ (10-19%) │
└──────────┘   └──────────┘
        │             │
        └──────┬──────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
┌──────────┐   ┌──────────┐
│ Shard 9  │   │ Shard 10 │
│ (80-89%) │   │ (90-99%) │
└──────────┘   └──────────┘
```

---

## API Design

### 1. Shorten URL

**Endpoint:** `POST /api/v1/shorten`

**Request:**
```json
{
  "long_url": "https://www.example.com/very/long/path",
  "expires_in_days": 365
}
```

**Response:**
```json
{
  "short_url": "abc1234",
  "long_url": "https://www.example.com/very/long/path",
  "expires_at": "2025-12-31T23:59:59Z"
}
```

**Implementation:**
```java
@PostMapping("/api/v1/shorten")
public ResponseEntity<ShortenResponse> shortenUrl(
        @RequestBody ShortenRequest request) {
    
    // Validate URL
    if (!isValidUrl(request.getLongUrl())) {
        return ResponseEntity.badRequest().build();
    }
    
    // Check if already exists
    String existingShortUrl = urlRepository.findByLongUrl(request.getLongUrl());
    if (existingShortUrl != null) {
        return ResponseEntity.ok(new ShortenResponse(existingShortUrl, request.getLongUrl()));
    }
    
    // Generate short URL
    String shortUrl = urlGenerator.generate(request.getLongUrl());
    
    // Save to database
    UrlMapping mapping = new UrlMapping(shortUrl, request.getLongUrl(), 
                                       calculateExpiration(request.getExpiresInDays()));
    urlRepository.save(mapping);
    
    // Cache it
    cacheService.set(shortUrl, request.getLongUrl(), 3600);
    
    return ResponseEntity.ok(new ShortenResponse(shortUrl, request.getLongUrl()));
}
```

---

### 2. Redirect URL

**Endpoint:** `GET /{short_url}`

**Response:**
```
HTTP/1.1 302 Found
Location: https://www.example.com/very/long/path
```

**Implementation:**
```java
@GetMapping("/{shortUrl}")
public ResponseEntity<Void> redirect(@PathVariable String shortUrl) {
    
    // Check cache first
    String longUrl = cacheService.get(shortUrl);
    if (longUrl != null) {
        // Update analytics asynchronously
        analyticsService.trackClick(shortUrl);
        return ResponseEntity.status(HttpStatus.FOUND)
            .header("Location", longUrl)
            .build();
    }
    
    // Cache miss - query database
    UrlMapping mapping = urlRepository.findByShortUrl(shortUrl);
    if (mapping == null || mapping.isExpired()) {
        return ResponseEntity.notFound().build();
    }
    
    // Cache it
    cacheService.set(shortUrl, mapping.getLongUrl(), 3600);
    
    // Update analytics
    analyticsService.trackClick(shortUrl);
    
    return ResponseEntity.status(HttpStatus.FOUND)
        .header("Location", mapping.getLongUrl())
        .build();
}
```

---

### 3. Get Analytics

**Endpoint:** `GET /api/v1/analytics/{short_url}`

**Response:**
```json
{
  "short_url": "abc1234",
  "long_url": "https://www.example.com/very/long/path",
  "click_count": 12345,
  "created_at": "2024-01-01T00:00:00Z",
  "top_countries": [
    {"country": "US", "clicks": 5000},
    {"country": "IN", "clicks": 3000}
  ]
}
```

---

## URL Generation Algorithm

### Option 1: Hash-based (Recommended)

```java
public class UrlGenerator {
    
    private static final String BASE62 = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
    private static final int SHORT_URL_LENGTH = 7;
    
    public String generate(String longUrl) {
        // 1. Hash the long URL
        String hash = md5(longUrl + System.currentTimeMillis());
        
        // 2. Take first 7 characters of hash
        String shortUrl = hash.substring(0, SHORT_URL_LENGTH);
        
        // 3. Check for collision
        while (urlRepository.exists(shortUrl)) {
            // Add salt and rehash
            hash = md5(longUrl + System.currentTimeMillis() + Math.random());
            shortUrl = hash.substring(0, SHORT_URL_LENGTH);
        }
        
        return shortUrl;
    }
    
    private String md5(String input) {
        // MD5 hash implementation
        MessageDigest md = MessageDigest.getInstance("MD5");
        byte[] hashBytes = md.digest(input.getBytes());
        return bytesToBase62(hashBytes);
    }
}
```

**Pros:**
- Deterministic (same URL → same short URL if exists)
- No database lookup needed for generation
- Fast

**Cons:**
- Possible collisions (handled by checking)

---

### Option 2: Counter-based

```java
public class UrlGenerator {
    
    private AtomicLong counter = new AtomicLong(0);
    
    public String generate(String longUrl) {
        long number = counter.incrementAndGet();
        return base62Encode(number);
    }
    
    private String base62Encode(long number) {
        StringBuilder sb = new StringBuilder();
        while (number > 0) {
            sb.append(BASE62.charAt((int)(number % 62)));
            number /= 62;
        }
        return sb.reverse().toString();
    }
}
```

**Pros:**
- No collisions
- Sequential (easier to manage)

**Cons:**
- Requires database lookup
- Predictable (security concern)
- Single point of failure (counter)

---

## Scaling the System

### 1. Horizontal Scaling

```
┌──────────────┐
│ Load         │
│ Balancer    │
└──────┬───────┘
       │
   ┌───┼───┬───┬───┐
   │   │   │   │   │
   ▼   ▼   ▼   ▼   ▼
┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
│ API │ │ API │ │ API │ │ API │ │ API │
│Srv 1│ │Srv 2│ │Srv 3│ │Srv 4│ │Srv N│
└─────┘ └─────┘ └─────┘ └─────┘ └─────┘
```

**Benefits:**
- Handle more traffic
- Fault tolerance
- Easy to add/remove servers

---

### 2. Database Scaling

**Read Replicas:**
```
┌──────────┐
│  Master  │  ← Writes
└────┬─────┘
     │
┌────┼────┬────┐
│    │    │    │
▼    ▼    ▼    ▼
┌──┐ ┌──┐ ┌──┐ ┌──┐
│R1│ │R2│ │R3│ │R4│  ← Reads
└──┘ └──┘ └──┘ └──┘
```

**Sharding:**
```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Shard 1  │  │ Shard 2  │  │ Shard 3  │
│ (0-33%)  │  │ (34-66%) │  │ (67-100%)│
└──────────┘  └──────────┘  └──────────┘
```

---

### 3. Caching Strategy

**Multi-Level Caching:**
```
┌─────────────────────────────────┐
│      Application Layer          │
└──────────────┬──────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
┌──────────┐   ┌──────────┐
│ L1 Cache │   │ L2 Cache │
│ (Local)  │   │ (Redis)  │
│ 1ms      │   │ 5ms      │
└──────────┘   └──────────┘
        │             │
        └──────┬───────┘
               │
               ▼
        ┌──────────┐
        │ Database │
        │ 50ms     │
        └──────────┘
```

**Cache Invalidation:**
- TTL-based: 1 hour for hot URLs
- LRU eviction: Remove least recently used
- Write-through: Update cache on write

---

## Trade-offs and Optimizations

### 1. Short URL Length

**6 Characters:**
- 62^6 = 56.8 billion URLs
- More collisions
- Shorter URLs (better UX)

**7 Characters:**
- 62^7 = 3.5 trillion URLs
- Fewer collisions
- Slightly longer URLs

**Decision:** Use 7 characters (better collision resistance)

---

### 2. Cache vs Database

**Cache Hit Rate:**
- 20% of URLs get 80% of traffic (80/20 rule)
- Cache top 20% = 80% cache hit rate
- Reduces database load by 80%

**Decision:** Cache hot URLs, query DB for cold URLs

---

### 3. Consistency vs Availability

**Strong Consistency:**
- All reads see latest data
- Slower (wait for replication)
- Better for analytics

**Eventual Consistency:**
- Faster reads
- May see stale data
- Acceptable for redirects

**Decision:** Eventual consistency for redirects, strong for analytics

---

### 4. Analytics Storage

**Option 1: Same Database**
- Simple
- Slows down redirects
- Database bloat

**Option 2: Separate Database**
- Fast redirects
- Complex
- More infrastructure

**Option 3: Asynchronous Processing**
- Fast redirects
- Eventual analytics
- Best of both worlds

**Decision:** Use message queue for async analytics processing

```
Redirect Request:
1. Redirect immediately (fast)
2. Send analytics event to queue (async)
3. Analytics service processes later
```

---

## Final Architecture

```
┌─────────────────────────────────────────┐
│           Client Layer                  │
│    (Web, Mobile, API Clients)          │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│         Load Balancer                   │
│    (Distributes traffic)                │
└──────────────┬──────────────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
┌──────────┐   ┌──────────┐
│ API      │   │ API      │
│ Server 1 │   │ Server 2 │
└────┬─────┘   └────┬─────┘
     │              │
     └──────┬───────┘
            │
     ┌──────┴──────┐
     │             │
     ▼             ▼
┌──────────┐ ┌──────────┐
│  Cache   │ │ Message  │
│ (Redis)  │ │  Queue   │
└──────────┘ └──────────┘
     │             │
     │             ▼
     │      ┌──────────┐
     │      │Analytics │
     │      │ Service  │
     │      └──────────┘
     │
     ▼
┌─────────────────────────────────────────┐
│         Database Layer                  │
│    ┌──────────┐      ┌──────────┐      │
│    │ Shard 1  │      │ Shard 2  │      │
│    └──────────┘      └──────────┘      │
│    ┌──────────┐      ┌──────────┐      │
│    │ Shard 3  │      │ Shard 4  │      │
│    └──────────┘      └──────────┘      │
└─────────────────────────────────────────┘
```

---

## Summary

**Key Design Decisions:**
1. ✅ Hash-based URL generation (fast, deterministic)
2. ✅ 7-character short URLs (good collision resistance)
3. ✅ Database sharding (horizontal scaling)
4. ✅ Multi-level caching (80% hit rate)
5. ✅ Asynchronous analytics (fast redirects)
6. ✅ Read replicas (scale reads)

**Capacity:**
- ✅ 1,200 writes/second
- ✅ 120,000 reads/second
- ✅ 40 TB storage
- ✅ 100 GB cache

**Next Steps:**
- Learn [Designing a Chat System](./14_DESIGNING_CHAT_SYSTEM.md)
- Understand [Designing a News Feed](./15_DESIGNING_NEWS_FEED.md)
- Practice [More System Designs](./README.md)

---

**References:**
- [bit.ly Architecture](https://bitly.com)
- [TinyURL Design](https://www.tinyurl.com)


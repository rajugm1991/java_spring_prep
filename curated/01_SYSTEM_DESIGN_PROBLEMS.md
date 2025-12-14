# System Design Problems - 50+ Real-World Problems

> **Curated from top tech companies and real production scenarios**

This guide contains 50+ real-world system design problems with comprehensive solutions, suitable for staff engineer interviews.

---

## Table of Contents

1. [Basic System Design Problems](#basic-system-design-problems)
2. [Intermediate System Design Problems](#intermediate-system-design-problems)
3. [Advanced System Design Problems](#advanced-system-design-problems)
4. [Expert-Level System Design Problems](#expert-level-system-design-problems)

---

## Basic System Design Problems

### Problem 1: Design a URL Shortener (bit.ly)

**Problem Statement:**
Design a URL shortener service like bit.ly that can shorten long URLs and redirect to original URLs.

**Requirements:**
- Shorten long URLs to short URLs (e.g., bit.ly/abc123)
- Redirect short URLs to original URLs
- Handle 100M URLs per day
- Short URLs should be unique
- Short URLs expire after 1 year

**Solution Approach:**

1. **Capacity Estimation:**
   - 100M URLs/day = ~1,160 URLs/second
   - Peak traffic: 5x = ~5,800 URLs/second
   - Storage: 100M URLs × 500 bytes = 50GB/year
   - With 5 years retention: 250GB

2. **API Design:**
   ```
   POST /api/v1/shorten
   {
     "longUrl": "https://example.com/very/long/url"
   }
   Response: {
     "shortUrl": "bit.ly/abc123"
   }
   
   GET /abc123
   Response: 301 Redirect to original URL
   ```

3. **Database Schema:**
   ```sql
   CREATE TABLE urls (
     id BIGINT PRIMARY KEY AUTO_INCREMENT,
     short_code VARCHAR(7) UNIQUE NOT NULL,
     long_url TEXT NOT NULL,
     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
     expires_at TIMESTAMP,
     click_count BIGINT DEFAULT 0,
     INDEX idx_short_code (short_code),
     INDEX idx_expires_at (expires_at)
   );
   ```

4. **Algorithm for Short Code Generation:**
   ```java
   // Base62 encoding (a-z, A-Z, 0-9)
   public class UrlShortener {
       private static final String BASE62 = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
       private static final int SHORT_CODE_LENGTH = 7;
       
       public String generateShortCode(long id) {
           StringBuilder code = new StringBuilder();
           while (id > 0) {
               code.append(BASE62.charAt((int)(id % 62)));
               id /= 62;
           }
           while (code.length() < SHORT_CODE_LENGTH) {
               code.append('a');
           }
           return code.reverse().toString();
       }
   }
   ```

5. **Architecture:**
   ```
   Client → Load Balancer → API Servers → Database
                              ↓
                          Cache (Redis)
   ```

**Trade-offs:**
- ✅ Simple and scalable
- ✅ Base62 encoding provides 62^7 = 3.5 trillion unique codes
- ⚠️ Database becomes bottleneck at scale (need sharding)
- ⚠️ Cache needed for hot URLs

**Follow-up Questions:**
- How to handle collisions?
- How to implement custom short URLs?
- How to track analytics (clicks, geolocation)?

---

### Problem 2: Design a Pastebin (pastebin.com)

**Problem Statement:**
Design a service where users can store text and retrieve it using a unique link.

**Requirements:**
- Store text content (max 10MB)
- Generate unique links
- Support expiration (1 hour to 1 year)
- 10M pastes per day
- Support syntax highlighting

**Solution Approach:**

1. **API Design:**
   ```
   POST /api/v1/paste
   {
     "content": "code here",
     "expires_in": 3600,
     "language": "java"
   }
   Response: {
     "pasteId": "abc123",
     "url": "pastebin.com/abc123"
   }
   
   GET /api/v1/paste/abc123
   Response: {
     "content": "code here",
     "language": "java",
     "created_at": "2024-01-01T00:00:00Z"
   }
   ```

2. **Storage:**
   - Hot data (recent pastes): Redis (TTL based on expiration)
   - Cold data: Object storage (S3) or Database
   - Metadata: SQL database (pasteId, expiration, language)

3. **Architecture:**
   ```
   Client → CDN → Load Balancer → API Servers
                                    ↓
                              Redis (hot) + S3 (cold)
   ```

**Trade-offs:**
- ✅ CDN for static content delivery
- ✅ Redis for fast retrieval of recent pastes
- ⚠️ Need cleanup job for expired pastes

---

### Problem 3: Design a Rate Limiter

**Problem Statement:**
Design a rate limiter that limits the number of requests a user can make per time window.

**Requirements:**
- Limit: 100 requests per minute per user
- Support multiple rate limit rules
- Distributed (multiple servers)
- Low latency (< 10ms)

**Solution Approach:**

1. **Algorithm: Token Bucket**
   ```java
   public class TokenBucketRateLimiter {
       private final RedisTemplate<String, String> redis;
       private final int maxTokens;
       private final int refillRate; // tokens per second
       
       public boolean allowRequest(String userId) {
           String key = "rate_limit:" + userId;
           long now = System.currentTimeMillis();
           
           // Get current tokens
           String tokenInfo = redis.get(key);
           if (tokenInfo == null) {
               // First request, initialize with max tokens
               redis.setex(key, 60, String.valueOf(maxTokens - 1));
               return true;
           }
           
           int tokens = Integer.parseInt(tokenInfo);
           if (tokens > 0) {
               redis.decr(key);
               return true;
           }
           
           return false; // Rate limit exceeded
       }
   }
   ```

2. **Distributed Rate Limiting:**
   - Use Redis with atomic operations
   - Sliding window log algorithm
   - Lua scripts for atomicity

**Trade-offs:**
- ✅ Redis provides distributed coordination
- ✅ Low latency
- ⚠️ Redis becomes single point of failure (use cluster)

---

## Intermediate System Design Problems

### Problem 4: Design a News Feed System (Facebook Feed)

**Problem Statement:**
Design a news feed system that shows posts from friends and pages a user follows.

**Requirements:**
- 1B users, 500M daily active users
- Each user follows 200 people on average
- Each user posts 2 times per day
- Feed should be ranked by relevance
- Feed generation latency < 200ms

**Solution Approach:**

1. **Approach: Pull vs Push**
   - **Pull (Fan-out on read)**: User requests feed, system fetches posts from all friends
     - Pros: Simple, real-time
     - Cons: Slow for users with many friends
   - **Push (Fan-out on write)**: When user posts, push to all followers' feeds
     - Pros: Fast feed retrieval
     - Cons: Expensive for celebrities (millions of followers)
   - **Hybrid**: Push for normal users, pull for celebrities

2. **Feed Generation:**
   ```
   User Request → Load Balancer → Feed Service
                                      ↓
                              Cache (Redis) - Pre-computed feed
                                      ↓
                              If cache miss:
                              - Fetch from friends' activity feed
                              - Rank by relevance (ML model)
                              - Cache result
   ```

3. **Ranking Algorithm:**
   - Factors: Post time, engagement (likes, comments), user preferences
   - ML model: Predict user engagement
   - Real-time features: Time since post, friend relationship strength

4. **Database Schema:**
   ```sql
   -- User posts
   CREATE TABLE posts (
     post_id BIGINT PRIMARY KEY,
     user_id BIGINT NOT NULL,
     content TEXT,
     created_at TIMESTAMP,
     INDEX idx_user_created (user_id, created_at)
   );
   
   -- User relationships
   CREATE TABLE friendships (
     user_id BIGINT,
     friend_id BIGINT,
     PRIMARY KEY (user_id, friend_id)
   );
   
   -- Pre-computed feed (for push model)
   CREATE TABLE user_feeds (
     user_id BIGINT,
     post_id BIGINT,
     score DECIMAL, -- relevance score
     created_at TIMESTAMP,
     PRIMARY KEY (user_id, post_id),
     INDEX idx_user_score (user_id, score DESC)
   );
   ```

**Trade-offs:**
- ✅ Hybrid approach balances performance and cost
- ✅ Caching improves latency
- ⚠️ Complex ranking algorithm
- ⚠️ Need to handle celebrity users differently

---

### Problem 5: Design a Chat System (WhatsApp)

**Problem Statement:**
Design a real-time chat system supporting 1B users with 50B messages per day.

**Requirements:**
- One-on-one and group chats
- Real-time message delivery (< 100ms)
- Message persistence
- Read receipts
- Typing indicators
- Support media (images, videos)

**Solution Approach:**

1. **Architecture:**
   ```
   Client → Load Balancer → API Gateway
                              ↓
                    ┌─────────┴─────────┐
                    ↓                   ↓
              Message Service      Presence Service
                    ↓                   ↓
              WebSocket Server    Redis (online status)
                    ↓
              Message Queue (Kafka)
                    ↓
              Database (Cassandra)
                    ↓
              Object Storage (S3) - Media
   ```

2. **Real-time Communication:**
   - WebSocket for bidirectional communication
   - Long polling as fallback
   - Message queue for reliable delivery

3. **Message Flow:**
   ```
   User A sends message
     ↓
   Message Service receives
     ↓
   Store in database (Cassandra)
     ↓
   Publish to Kafka topic (user-b's-messages)
     ↓
   If User B is online:
     - Push via WebSocket
   Else:
     - Store for later delivery
   ```

4. **Database Design:**
   ```sql
   -- Messages table (Cassandra - partitioned by chat_id)
   CREATE TABLE messages (
     chat_id BIGINT,
     message_id TIMEUUID,
     sender_id BIGINT,
     content TEXT,
     message_type VARCHAR, -- text, image, video
     created_at TIMESTAMP,
     PRIMARY KEY (chat_id, message_id)
   ) WITH CLUSTERING ORDER BY (message_id DESC);
   ```

5. **Presence Service:**
   - Redis: Store online users
   - Key: `presence:user_id`, Value: `online`, TTL: 30 seconds
   - Heartbeat: Client sends ping every 20 seconds

**Trade-offs:**
- ✅ WebSocket provides low latency
- ✅ Kafka ensures message delivery
- ✅ Cassandra handles high write throughput
- ⚠️ Complex to handle offline users
- ⚠️ Need to handle message ordering

---

### Problem 6: Design a Search Engine (Google Search)

**Problem Statement:**
Design a search engine that can index billions of web pages and return relevant results in < 100ms.

**Requirements:**
- Index 100B web pages
- 5B searches per day
- Search latency < 100ms
- Support ranking by relevance
- Support autocomplete

**Solution Approach:**

1. **Components:**
   - **Crawler**: Fetches web pages
   - **Indexer**: Builds inverted index
   - **Ranker**: Ranks results by relevance
   - **Query Service**: Handles search queries

2. **Inverted Index:**
   ```
   Word → [Document IDs where word appears]
   
   Example:
   "java" → [doc1, doc5, doc10, ...]
   "spring" → [doc1, doc3, doc10, ...]
   
   Query: "java spring"
   - Find docs containing both "java" AND "spring"
   - Intersection: [doc1, doc10]
   ```

3. **Architecture:**
   ```
   User Query → Load Balancer → Query Service
                                      ↓
                              Inverted Index (Distributed)
                                      ↓
                              Ranker (ML model)
                                      ↓
                              Cache (Redis) - Popular queries
   ```

4. **Ranking Algorithm (PageRank + ML):**
   - PageRank: Authority score based on links
   - TF-IDF: Term frequency, inverse document frequency
   - ML features: Click-through rate, dwell time, bounce rate
   - Real-time: Freshness, user location, device type

5. **Autocomplete:**
   - Trie data structure
   - Cache popular queries
   - Update based on trending searches

**Trade-offs:**
- ✅ Distributed index for scalability
- ✅ Caching improves latency
- ⚠️ Indexing is expensive (batch processing)
- ⚠️ Ranking algorithm is complex

---

## Advanced System Design Problems

### Problem 7: Design a Distributed Cache (Redis-like)

**Problem Statement:**
Design a distributed caching system that can handle 1M requests per second with < 1ms latency.

**Requirements:**
- Support get/set operations
- TTL (time-to-live) support
- Replication for high availability
- Consistent hashing for sharding
- Handle node failures

**Solution Approach:**

1. **Consistent Hashing:**
   ```java
   public class ConsistentHash {
       private final TreeMap<Long, String> ring = new TreeMap<>();
       private final int replicas = 150; // virtual nodes
       
       public void addNode(String node) {
           for (int i = 0; i < replicas; i++) {
               long hash = hash(node + ":" + i);
               ring.put(hash, node);
           }
       }
       
       public String getNode(String key) {
           if (ring.isEmpty()) return null;
           long hash = hash(key);
           Map.Entry<Long, String> entry = ring.ceilingEntry(hash);
           return entry != null ? entry.getValue() : ring.firstEntry().getValue();
       }
   }
   ```

2. **Architecture:**
   ```
   Client → Load Balancer → Cache Proxy
                              ↓
                    Consistent Hash Ring
                    ↓         ↓         ↓
                  Node1     Node2     Node3
                    ↓         ↓         ↓
                  Replica1 Replica2 Replica3
   ```

3. **Replication Strategy:**
   - Master-Slave: Write to master, replicate to slaves
   - Read from slaves for load distribution
   - Failover: Promote slave to master on failure

4. **Cache Eviction:**
   - LRU (Least Recently Used)
   - LFU (Least Frequently Used)
   - TTL-based expiration

**Trade-offs:**
- ✅ Consistent hashing provides even distribution
- ✅ Replication ensures availability
- ⚠️ Cache invalidation is complex
- ⚠️ Need to handle split-brain scenarios

---

### Problem 8: Design a Video Streaming Service (Netflix)

**Problem Statement:**
Design a video streaming service that can serve 200M users with 1B hours of video per day.

**Requirements:**
- Support multiple video qualities (240p to 4K)
- Adaptive bitrate streaming
- Low latency (< 2 seconds to start)
- Support live streaming
- Handle peak traffic (evening hours)

**Solution Approach:**

1. **Video Processing Pipeline:**
   ```
   Upload → Transcoding Service → Multiple Qualities
                                      ↓
                              Object Storage (S3)
                                      ↓
                              CDN (CloudFront)
   ```

2. **Adaptive Bitrate Streaming:**
   - HLS (HTTP Live Streaming) or DASH
   - Video split into chunks (2-10 seconds)
   - Multiple quality versions of each chunk
   - Client selects quality based on bandwidth

3. **Architecture:**
   ```
   Client → CDN (Edge Locations)
              ↓ (cache miss)
           Origin Server
              ↓
           Object Storage
   ```

4. **CDN Strategy:**
   - Hot content: Cached at edge locations
   - Cold content: Fetched from origin
   - Pre-warming: Popular content pre-cached

5. **Live Streaming:**
   - RTMP ingest → Transcoding → HLS segments
   - Low latency: WebRTC for real-time streaming
   - Scale: Multiple transcoding servers

**Trade-offs:**
- ✅ CDN reduces latency and bandwidth costs
- ✅ Adaptive bitrate improves user experience
- ⚠️ Transcoding is expensive (GPU instances)
- ⚠️ Storage costs are high

---

### Problem 9: Design a Payment System (Stripe)

**Problem Statement:**
Design a payment processing system that handles 1M transactions per day with 99.99% availability.

**Requirements:**
- Process credit card payments
- Support multiple payment methods
- Fraud detection
- Transaction history
- Refund support
- PCI DSS compliance

**Solution Approach:**

1. **Architecture:**
   ```
   Merchant → Payment Gateway → Payment Processor
                                      ↓
                              Fraud Detection Service
                                      ↓
                              Bank/Card Network
   ```

2. **Payment Flow:**
   ```
   1. Merchant sends payment request
   2. Payment Gateway validates request
   3. Fraud Detection checks for suspicious activity
   4. Payment Processor routes to bank
   5. Bank authorizes/declines
   6. Response sent back to merchant
   7. Transaction recorded in database
   ```

3. **Database Design:**
   ```sql
   CREATE TABLE transactions (
     transaction_id BIGINT PRIMARY KEY,
     merchant_id BIGINT NOT NULL,
     amount DECIMAL(10,2) NOT NULL,
     currency VARCHAR(3) NOT NULL,
     payment_method VARCHAR(50),
     status VARCHAR(20), -- pending, approved, declined, refunded
     created_at TIMESTAMP,
     INDEX idx_merchant_created (merchant_id, created_at)
   );
   ```

4. **Idempotency:**
   - Use idempotency keys to prevent duplicate charges
   - Store idempotency key → transaction mapping

5. **Fraud Detection:**
   - ML model: Anomaly detection
   - Features: Amount, location, device, user history
   - Real-time scoring: < 100ms

**Trade-offs:**
- ✅ Idempotency prevents duplicate charges
- ✅ Fraud detection protects merchants
- ⚠️ PCI DSS compliance is complex
- ⚠️ Need to handle bank failures gracefully

---

## Expert-Level System Design Problems

### Problem 10: Design a Distributed File System (Google File System)

**Problem Statement:**
Design a distributed file system that can store petabytes of data across thousands of machines.

**Requirements:**
- Store large files (GB to TB)
- High availability (99.9%)
- Fault tolerance
- Consistency guarantees
- Support concurrent reads/writes

**Solution Approach:**

1. **Architecture:**
   ```
   Client → NameNode (Metadata)
              ↓
           DataNodes (Actual Data)
   ```

2. **Components:**
   - **NameNode**: Stores metadata (file names, locations, permissions)
   - **DataNodes**: Store actual file chunks
   - **Replication**: Each chunk replicated 3x across different nodes

3. **File Chunking:**
   - Large files split into chunks (64MB default)
   - Each chunk stored on different DataNode
   - Replication for fault tolerance

4. **Consistency:**
   - Write: Write to primary, replicate to secondaries
   - Read: Read from any replica
   - Metadata: Strongly consistent (NameNode)

**Trade-offs:**
- ✅ Replication provides fault tolerance
- ✅ Chunking enables parallel processing
- ⚠️ NameNode is single point of failure (need standby)
- ⚠️ Network bandwidth is bottleneck

---

### Problem 11: Design a Distributed Lock Service

**Problem Statement:**
Design a distributed lock service that can coordinate access to shared resources across multiple servers.

**Requirements:**
- Mutual exclusion (only one process can hold lock)
- Deadlock prevention
- Automatic expiration
- High availability
- Low latency

**Solution Approach:**

1. **Redis-based Lock:**
   ```java
   public class DistributedLock {
       private final RedisTemplate<String, String> redis;
       
       public boolean acquireLock(String resource, String lockId, int ttlSeconds) {
           String key = "lock:" + resource;
           Boolean acquired = redis.opsForValue()
               .setIfAbsent(key, lockId, Duration.ofSeconds(ttlSeconds));
           return Boolean.TRUE.equals(acquired);
       }
       
       public void releaseLock(String resource, String lockId) {
           String key = "lock:" + resource;
           String script = 
               "if redis.call('get', KEYS[1]) == ARGV[1] then " +
               "return redis.call('del', KEYS[1]) " +
               "else return 0 end";
           redis.execute(new DefaultRedisScript<>(script, Long.class),
               Collections.singletonList(key), lockId);
       }
   }
   ```

2. **Zookeeper-based Lock:**
   - Create ephemeral znode
   - Smallest znode gets lock
   - Auto-release on disconnect

**Trade-offs:**
- ✅ Redis: Simple, fast
- ✅ Zookeeper: Stronger guarantees
- ⚠️ Need to handle lock expiration
- ⚠️ Network partitions can cause issues

---

## Summary

These 10 problems cover:
- ✅ Basic systems (URL shortener, pastebin)
- ✅ Intermediate systems (news feed, chat, search)
- ✅ Advanced systems (cache, video streaming, payments)
- ✅ Expert systems (file system, distributed locks)

**Key Patterns:**
- Load balancing
- Caching strategies
- Database sharding
- Message queues
- CDN usage
- Replication
- Consistent hashing

**Next Steps:**
- Practice each problem
- Draw architecture diagrams
- Consider trade-offs
- Think about scale

---

*This guide is continuously updated with new problems and solutions.*


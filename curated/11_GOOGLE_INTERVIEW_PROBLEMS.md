# Google Interview Problems - Real Questions & Solutions

> **Curated from actual Google staff engineer interviews**

This guide contains real system design and low-level design problems asked in Google interviews, with comprehensive solutions.

---

## Table of Contents

1. [System Design Problems](#system-design-problems)
2. [Low-Level Design Problems](#low-level-design-problems)
3. [Distributed Systems Problems](#distributed-systems-problems)
4. [Interview Tips](#interview-tips)

---

## System Design Problems

### Problem 1: Design Google Search

**Problem Statement:**
Design a search engine that can index billions of web pages and return relevant results in < 100ms.

**Requirements:**
- Index 100 billion web pages
- 5 billion searches per day
- Search latency < 100ms
- Support ranking by relevance
- Support autocomplete

**Solution Approach:**

1. **Capacity Estimation:**
   - 5B searches/day = ~58K searches/second
   - Peak: 5x = ~290K searches/second
   - Storage: 100B pages × 10KB = 1PB

2. **Components:**
   - **Crawler**: Fetches web pages
   - **Indexer**: Builds inverted index
   - **Ranker**: Ranks results by relevance
   - **Query Service**: Handles search queries

3. **Architecture:**
   ```
   User Query → Load Balancer → Query Service
                                      ↓
                              Inverted Index (Distributed)
                                      ↓
                              Ranker (ML model)
                                      ↓
                              Cache (Popular queries)
   ```

4. **Inverted Index:**
   ```
   Word → [Document IDs where word appears]
   
   Example:
   "java" → [doc1, doc5, doc10, ...]
   "spring" → [doc1, doc3, doc10, ...]
   
   Query: "java spring"
   - Find docs containing both "java" AND "spring"
   - Intersection: [doc1, doc10]
   ```

5. **Ranking Algorithm:**
   - **PageRank**: Authority score based on links
   - **TF-IDF**: Term frequency, inverse document frequency
   - **ML Features**: Click-through rate, dwell time, bounce rate
   - **Real-time**: Freshness, user location, device type

6. **Autocomplete:**
   - Trie data structure
   - Cache popular queries
   - Update based on trending searches

**Follow-up Questions:**
- How to handle misspellings? (Fuzzy matching, edit distance)
- How to rank results? (PageRank + ML model)
- How to handle real-time updates? (Incremental indexing)

---

### Problem 2: Design Google Maps

**Problem Statement:**
Design a mapping service that can provide directions, real-time traffic, and location search.

**Requirements:**
- 1 billion users
- 100 million route requests per day
- Route calculation < 1 second
- Real-time traffic updates
- Support multiple transportation modes

**Solution Approach:**

1. **Components:**
   - **Map Data Storage**: Road network, POIs
   - **Route Calculation**: Shortest path algorithm
   - **Traffic Service**: Real-time traffic data
   - **Geocoding**: Address to coordinates

2. **Architecture:**
   ```
   User Request → Load Balancer → Route Service
                                      ↓
                              Map Data (Pre-processed)
                                      ↓
                              Traffic Service (Real-time)
                                      ↓
                              Route Calculation (Dijkstra/A*)
   ```

3. **Route Calculation:**
   - **Algorithm**: A* algorithm (heuristic search)
   - **Pre-processing**: Contraction hierarchies for faster queries
   - **Caching**: Cache popular routes
   - **Parallel**: Calculate multiple routes in parallel

4. **Traffic Data:**
   - **Sources**: GPS data from users, traffic sensors
   - **Processing**: Real-time aggregation
   - **Storage**: Time-series database
   - **Updates**: Every 5 minutes

5. **Map Data:**
   - **Storage**: Graph database (nodes = intersections, edges = roads)
   - **Partitioning**: Geographic sharding
   - **Updates**: Incremental updates

**Follow-up Questions:**
- How to handle real-time traffic? (GPS data aggregation)
- How to calculate routes quickly? (Pre-processing, caching)
- How to handle map updates? (Incremental updates)

---

### Problem 3: Design Gmail

**Problem Statement:**
Design an email service that can handle billions of emails with fast search and organization.

**Requirements:**
- 1.5 billion users
- 300 billion emails per day
- Search latency < 100ms
- Support labels, filters, threads
- 15GB storage per user

**Solution Approach:**

1. **Components:**
   - **Email Storage**: Store emails
   - **Search Index**: Full-text search
   - **Threading**: Group related emails
   - **Spam Filter**: ML-based spam detection

2. **Architecture:**
   ```
   User Request → Load Balancer → Email Service
                                      ↓
                              Email Storage (Distributed)
                                      ↓
                              Search Index (Elasticsearch)
                                      ↓
                              Spam Filter (ML model)
   ```

3. **Email Storage:**
   - **Partitioning**: Shard by user_id
   - **Replication**: 3x replication for availability
   - **Compression**: Compress attachments
   - **Archival**: Move old emails to cold storage

4. **Search:**
   - **Index**: Inverted index for full-text search
   - **Partitioning**: Shard by user_id
   - **Caching**: Cache popular searches
   - **Ranking**: Rank by relevance, date

5. **Threading:**
   - **Algorithm**: Group by subject, references header
   - **Storage**: Store thread relationships
   - **Updates**: Incremental updates

**Follow-up Questions:**
- How to handle spam? (ML-based spam filter)
- How to search quickly? (Inverted index, caching)
- How to handle attachments? (Object storage, compression)

---

## Low-Level Design Problems

### Problem 4: Design a Distributed Lock

**Problem Statement:**
Design a distributed lock service that can coordinate access to shared resources across multiple servers.

**Requirements:**
- Mutual exclusion (only one process can hold lock)
- Deadlock prevention
- Automatic expiration
- High availability
- Low latency (< 10ms)

**Solution Approach:**

1. **Redis-based Implementation:**
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

2. **Zookeeper-based Implementation:**
   - Create ephemeral znode
   - Smallest znode gets lock
   - Auto-release on disconnect

3. **Challenges:**
   - **Deadlock Prevention**: TTL on locks
   - **Lock Expiration**: Renew lock before expiration
   - **Network Partitions**: Use quorum-based approach

**Follow-up Questions:**
- How to handle lock expiration? (Renewal mechanism)
- How to prevent deadlocks? (TTL, timeout)
- How to handle network partitions? (Quorum-based)

---

### Problem 5: Design a Rate Limiter

**Problem Statement:**
Design a rate limiter that limits the number of requests a user can make per time window.

**Requirements:**
- Limit: 100 requests per minute per user
- Support multiple rate limit rules
- Distributed (multiple servers)
- Low latency (< 10ms)

**Solution Approach:**

1. **Token Bucket Algorithm:**
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

2. **Sliding Window Log:**
   - Store timestamps of requests
   - Count requests in time window
   - Remove old timestamps

3. **Distributed Rate Limiting:**
   - Use Redis with atomic operations
   - Lua scripts for atomicity
   - Consistent hashing for sharding

**Follow-up Questions:**
- How to handle distributed rate limiting? (Redis, consistent hashing)
- How to support multiple rules? (Separate keys per rule)
- How to reduce memory usage? (Sliding window counter)

---

## Distributed Systems Problems

### Problem 6: Design a Distributed Cache

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

3. **Replication:**
   - Master-Slave: Write to master, replicate to slaves
   - Read from slaves for load distribution
   - Failover: Promote slave to master on failure

4. **Cache Eviction:**
   - LRU (Least Recently Used)
   - LFU (Least Frequently Used)
   - TTL-based expiration

**Follow-up Questions:**
- How to handle cache invalidation? (Event-based, TTL)
- How to handle split-brain? (Quorum-based)
- How to reduce memory usage? (Compression, eviction)

---

### Problem 7: Design a Distributed File System

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

**Follow-up Questions:**
- How to handle NameNode failure? (Standby NameNode)
- How to ensure consistency? (Quorum-based writes)
- How to handle network partitions? (CAP theorem trade-offs)

---

## Interview Tips

### What Google Looks For

1. **Problem-Solving**
   - Break down complex problems
   - Think systematically
   - Consider edge cases

2. **System Design**
   - Scalability thinking
   - Trade-off analysis
   - Real-world experience

3. **Communication**
   - Explain clearly
   - Ask clarifying questions
   - Discuss alternatives

4. **Technical Depth**
   - Deep understanding of systems
   - Knowledge of distributed systems
   - Experience with scale

### Common Mistakes to Avoid

1. **Jumping to Solution**
   - Always clarify requirements first
   - Ask about scale, constraints
   - Understand the problem

2. **Ignoring Scale**
   - Consider scale from the beginning
   - Estimate capacity
   - Design for growth

3. **No Trade-offs**
   - Every design has trade-offs
   - Discuss pros and cons
   - Justify choices

4. **No Alternatives**
   - Consider multiple approaches
   - Discuss alternatives
   - Explain why you chose one

### Preparation Tips

1. **Practice Problems**
   - Practice system design problems
   - Draw architecture diagrams
   - Explain your thinking

2. **Study Systems**
   - Study real systems (Google Search, Gmail)
   - Understand how they work
   - Learn from their design

3. **Mock Interviews**
   - Practice with peers
   - Get feedback
   - Improve communication

4. **Stay Updated**
   - Read engineering blogs
   - Follow tech news
   - Learn new technologies

---

## Summary

Google interview problems focus on:

✅ **Scalability**: Design for billions of users
✅ **Performance**: Low latency, high throughput
✅ **Distributed Systems**: Consistency, availability
✅ **Real-World**: Actual Google products
✅ **Problem-Solving**: Break down complex problems

**Key Takeaways:**
- Clarify requirements first
- Think about scale
- Discuss trade-offs
- Consider alternatives
- Communicate clearly

---

*This guide is continuously updated with new Google interview problems.*


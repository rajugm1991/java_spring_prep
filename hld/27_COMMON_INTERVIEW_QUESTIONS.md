# Common Interview Questions

## Table of Contents
1. [Top 50 System Design Questions](#top-50-system-design-questions)
2. [Design Approaches](#design-approaches)
3. [Common Patterns](#common-patterns)
4. [Interview Tips](#interview-tips)

---

## Top 50 System Design Questions

### Beginner Level

1. **Design a URL Shortener (bit.ly)**
   - [Complete Design](./13_DESIGNING_URL_SHORTENER.md)

2. **Design a Pastebin**
   - Similar to URL shortener
   - Text storage instead of URLs

3. **Design a Rate Limiter**
   - [See Guide](../imp/RATE_LIMITING_COMPLETE_GUIDE.md)

4. **Design a Distributed Cache**
   - [See Guide](../imp/REDIS_CACHING_GUIDE.md)

5. **Design a Counter Service**
   - Real-time counters
   - High write throughput

---

### Intermediate Level

6. **Design a Chat System (WhatsApp)**
   - [Complete Design](./14_DESIGNING_CHAT_SYSTEM.md)

7. **Design a News Feed (Facebook)**
   - Feed generation
   - Real-time updates

8. **Design a Search Engine (Google)**
   - Web crawling
   - Indexing
   - Ranking

9. **Design a Video Streaming Service (Netflix)**
   - Video encoding
   - CDN distribution
   - Adaptive bitrate

10. **Design a Payment System (Stripe)**
    - Transaction processing
    - Fraud detection
    - Idempotency

---

### Advanced Level

11. **Design Twitter**
    - Tweets, followers, timeline
    - Real-time feed

12. **Design Uber**
    - Ride matching
    - Real-time location
    - Pricing

13. **Design Instagram**
    - Photo storage
    - Feed generation
    - Stories

14. **Design YouTube**
    - Video upload
    - Streaming
    - Recommendations

15. **Design Amazon**
    - Product catalog
    - Shopping cart
    - Recommendations

---

## Design Approaches

### Question 1: Design Twitter

**Requirements:**
- Post tweets (140 characters)
- Follow users
- View timeline (home feed)
- View user profile

**Approach:**

1. **Clarify Requirements:**
   - 300M users
   - 200M daily active
   - 100 tweets/user/day
   - Read/write ratio: 100:1

2. **Capacity Estimation:**
   - 20B tweets/day
   - 230K QPS
   - 1.75 PB/year storage

3. **High-Level Design:**
   ```
   Client → Load Balancer → API Servers
                              ↓
                         ┌────┴────┐
                         │        │
                      Cache      Database
                      (Redis)   (Sharded)
   ```

4. **Detailed Design:**
   - Feed generation (fan-out)
   - Database sharding
   - Caching strategy

---

### Question 2: Design Uber

**Requirements:**
- Match riders with drivers
- Real-time location tracking
- Pricing calculation
- Payment processing

**Approach:**

1. **Clarify Requirements:**
   - 100M users
   - 10M daily active
   - 2 rides/user/day
   - Real-time matching

2. **Capacity Estimation:**
   - 20M rides/day
   - 231 QPS
   - 7.3 TB/year storage

3. **High-Level Design:**
   ```
   Client → API Gateway → Services
                            ↓
                    ┌───────┼───────┐
                    │       │       │
                Matching  Location Payment
   ```

4. **Detailed Design:**
   - Geospatial indexing
   - Real-time matching
   - Location updates

---

## Common Patterns

### Pattern 1: Feed Generation

**Pull Model:**
```
User requests feed → Query all followed users → Aggregate
```

**Push Model:**
```
User posts → Write to all followers' feeds
```

**Hybrid:**
```
Celebrities: Pull
Regular users: Push
```

---

### Pattern 2: Real-Time Systems

**WebSocket:**
```
Client ↔ WebSocket Server ↔ Services
```

**Long Polling:**
```
Client → Server (wait) → Response
```

**Server-Sent Events:**
```
Server → Client (push updates)
```

---

### Pattern 3: Search Systems

**Inverted Index:**
```
Word → [Document IDs]
"cat" → [1, 5, 10]
```

**Ranking:**
```
PageRank, TF-IDF, Machine Learning
```

---

## Interview Tips

### 1. **Ask Questions First**

Don't assume requirements. Ask:
- Scale?
- Features?
- Constraints?
- Performance?

---

### 2. **Start Simple**

Begin with basic design, then optimize:
- Single server → Multiple servers
- Single database → Sharded database
- No cache → Add cache

---

### 3. **Think Out Loud**

Explain your thought process:
"I'm considering two approaches. Let me think through the trade-offs..."

---

### 4. **Draw Diagrams**

Visualize your design:
"Let me draw the architecture..."

---

### 5. **Discuss Trade-offs**

Explain pros and cons:
"This approach is faster but uses more memory..."

---

## Summary

**Key Questions:**
1. ✅ URL Shortener, Chat System, News Feed
2. ✅ Twitter, Uber, Instagram
3. ✅ Search Engine, Video Streaming, Payment System

**Approach:**
1. ✅ Clarify requirements
2. ✅ Estimate capacity
3. ✅ Design architecture
4. ✅ Discuss trade-offs

**Next Steps:**
- Practice [Interview Process](./25_SYSTEM_DESIGN_INTERVIEW_PROCESS.md)
- Review [Back-of-the-Envelope](./26_BACK_OF_THE_ENVELOPE.md)
- Study [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [System Design Questions](https://algomaster.io/learn/system-design)
- [Interview Preparation](https://www.geeksforgeeks.org/system-design/)

# Designing a News Feed (Facebook)

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Requirements Gathering](#requirements-gathering)
3. [Capacity Estimation](#capacity-estimation)
4. [Feed Generation Strategies](#feed-generation-strategies)
5. [High-Level Design](#high-level-design)
6. [Detailed Design](#detailed-design)
7. [Scaling the System](#scaling-the-system)

---

## Problem Statement

Design a social media news feed system like Facebook that:
- Shows posts from friends
- Updates in real-time
- Supports ranking/algorithmic feed
- Handles billions of users

---

## Requirements Gathering

### Functional Requirements

1. **Feed Generation**
   - Show posts from followed users
   - Rank posts by relevance
   - Support pagination

2. **Real-Time Updates**
   - New posts appear in feed
   - Like/comment updates

3. **Features**
   - Like, comment, share
   - Media posts (images, videos)

### Non-Functional Requirements

1. **Scale**
   - 2B users
   - 1B daily active users
   - 500 posts/user/day
   - 200 friends/user (average)

2. **Performance**
   - Feed load: < 200ms
   - Real-time updates: < 1s

---

## Capacity Estimation

### Traffic Estimates

**Posts:**
```
1B users × 500 posts/day = 500B posts/day
= 5.8M posts/second
```

**Feed Reads:**
```
1B users × 5 feed reads/day = 5B reads/day
= 57,870 reads/second ≈ 60K reads/second
```

**Storage:**
```
500B posts/day × 1 KB = 500 TB/day
Retention: 1 year = 182.5 PB
```

---

## Feed Generation Strategies

### Strategy 1: Pull Model (Fan-out on Read)

**How it works:**
```
User requests feed:
1. Get list of followed users
2. Query posts from each user
3. Aggregate and rank
4. Return feed
```

**Pros:**
- Simple to implement
- Real-time data
- No pre-computation

**Cons:**
- Slow for users with many friends
- High database load
- Doesn't scale

**Use Case:** Small scale, few friends

---

### Strategy 2: Push Model (Fan-out on Write)

**How it works:**
```
User posts:
1. Save post to database
2. Get list of followers
3. Write post to each follower's feed
4. Store in feed cache
```

**Pros:**
- Fast feed reads (pre-computed)
- Scales reads well
- Good for active users

**Cons:**
- Slow writes (fan-out to many followers)
- Storage overhead
- Problem for celebrities (millions of followers)

**Use Case:** Regular users, active feeds

---

### Strategy 3: Hybrid Model (Recommended)

**How it works:**
```
Regular users (< 1000 followers): Push model
Celebrities (> 1000 followers): Pull model

User requests feed:
1. Get pre-computed feed (push)
2. Merge with celebrity posts (pull)
3. Rank and return
```

**Pros:**
- Best of both worlds
- Handles celebrities
- Scales well

**Cons:**
- More complex
- Two code paths

**Use Case:** Production systems (Facebook, Twitter)

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
│ Feed Service │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│Feed │ │Post │
│Cache│ │Svc  │
└─────┘ └─────┘
       │
       ▼
┌──────────┐
│ Database │
└──────────┘
```

### Components

1. **Feed Service:** Generate and serve feeds
2. **Post Service:** Handle posts
3. **Feed Cache:** Store pre-computed feeds
4. **Ranking Service:** Rank posts by relevance

---

## Detailed Design

### 1. Post Creation Flow

```
┌─────────────────────────────────────┐
│ 1. User creates post                │
│    Client → Post Service            │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 2. Save post to database            │
│    - Store post content             │
│    - Generate post ID               │
└──────────────┬──────────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
┌──────────┐   ┌──────────┐
│ Get      │   │ Check    │
│Followers │   │Follower  │
│          │   │Count     │
└────┬─────┘   └────┬─────┘
     │              │
     │         ┌────┴────┐
     │         │          │
     │         ▼          ▼
     │    ┌────────┐ ┌────────┐
     │    │< 1000  │ │> 1000  │
     │    │Regular │ │Celebrity│
     │    └───┬────┘ └───┬────┘
     │        │          │
     │        ▼          │
     │   ┌──────────┐    │
     │   │ Fan-out  │    │
     │   │ to feeds │    │
     │   └──────────┘    │
     │                   │
     └──────────┬────────┘
                │
                ▼
┌─────────────────────────────────────┐
│ 3. Update feed cache                │
│    - Regular users: Pre-compute    │
│    - Celebrities: Skip (pull later)│
└─────────────────────────────────────┘
```

### 2. Feed Retrieval Flow

```
┌─────────────────────────────────────┐
│ 1. User requests feed               │
│    Client → Feed Service            │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 2. Get pre-computed feed            │
│    - From feed cache (push model)   │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 3. Get celebrity posts              │
│    - Query posts from celebrities   │
│    - Pull model                     │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 4. Merge and rank                   │
│    - Combine push + pull            │
│    - Rank by relevance              │
│    - Apply filters                  │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 5. Return feed                      │
│    - Paginated results              │
│    - Cached for future requests     │
└─────────────────────────────────────┘
```

---

## Database Design

### Schema

```sql
CREATE TABLE posts (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    content TEXT,
    media_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user_created (user_id, created_at)
);

CREATE TABLE user_feeds (
    user_id BIGINT,
    post_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, post_id),
    INDEX idx_user_created (user_id, created_at)
);

CREATE TABLE follows (
    follower_id BIGINT,
    followee_id BIGINT,
    PRIMARY KEY (follower_id, followee_id)
);
```

---

## Ranking Algorithm

### Factors

1. **Recency:** Newer posts ranked higher
2. **Engagement:** Likes, comments, shares
3. **Relationship:** Close friends ranked higher
4. **Content Type:** Videos ranked higher than text

### Example

```java
public double calculateScore(Post post, User user) {
    double score = 0;
    
    // Recency (decay over time)
    long hoursAgo = (System.currentTimeMillis() - post.getCreatedAt()) / 3600000;
    score += 100 / (1 + hoursAgo);
    
    // Engagement
    score += post.getLikes() * 2;
    score += post.getComments() * 3;
    score += post.getShares() * 5;
    
    // Relationship strength
    double relationshipScore = getRelationshipScore(user, post.getUserId());
    score *= relationshipScore;
    
    return score;
}
```

---

## Scaling the System

### 1. Database Sharding

```
Shard by user_id:
hash(user_id) % number_of_shards = shard_number
```

### 2. Feed Cache

```
Cache top 100 posts per user
TTL: 5 minutes
Reduce database load by 90%
```

### 3. CDN for Media

```
Images/Videos served from CDN
Reduce origin server load
```

---

## Summary

**Key Design Decisions:**
1. ✅ **Hybrid Model:** Push for regular, pull for celebrities
2. ✅ **Feed Cache:** Pre-computed feeds
3. ✅ **Ranking:** Multi-factor algorithm
4. ✅ **Sharding:** Scale database
5. ✅ **CDN:** Media delivery

**Next Steps:**
- Learn [Designing Video Streaming](./16_DESIGNING_VIDEO_STREAMING.md)
- Understand [Event-Driven Architecture](./08_EVENT_DRIVEN_ARCHITECTURE.md)
- Practice [More System Designs](./README.md)

---

**References:**
- [Facebook Architecture](https://www.facebook.com)
- [Feed Generation Patterns](https://algomaster.io/learn/system-design)

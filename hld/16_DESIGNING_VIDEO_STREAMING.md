# Designing a Video Streaming Service (Netflix)

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Requirements Gathering](#requirements-gathering)
3. [Capacity Estimation](#capacity-estimation)
4. [High-Level Design](#high-level-design)
5. [Video Processing Pipeline](#video-processing-pipeline)
6. [CDN Architecture](#cdn-architecture)
7. [Scaling the System](#scaling-the-system)

---

## Problem Statement

Design a video streaming service like Netflix that:
- Streams videos to users
- Supports multiple devices
- Handles millions of concurrent streams
- Provides recommendations

---

## Requirements Gathering

### Functional Requirements

1. **Video Streaming**
   - Upload videos
   - Encode videos (multiple qualities)
   - Stream to users
   - Adaptive bitrate streaming

2. **User Features**
   - Search videos
   - Watch history
   - Recommendations
   - Multiple profiles

### Non-Functional Requirements

1. **Scale**
   - 200M users
   - 100M daily active
   - 2 hours watch time/user/day
   - Peak: 50M concurrent streams

2. **Performance**
   - Video start: < 2 seconds
   - No buffering
   - Adaptive quality

---

## Capacity Estimation

### Traffic Estimates

**Concurrent Streams:**
```
Peak: 50M concurrent streams
Average bitrate: 5 Mbps
Total bandwidth: 50M × 5 Mbps = 250 Tbps
```

**Storage:**
```
Videos: 10,000 titles
Average: 2 hours, 5 Mbps
Per video: 2 hours × 5 Mbps = 4.5 GB
Total: 10,000 × 4.5 GB = 45 TB (one quality)

Multiple qualities (4K, 1080p, 720p, 480p):
45 TB × 4 = 180 TB per title
10,000 titles × 180 TB = 1.8 PB
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
│   CDN        │
│  (Edge)      │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Video        │
│ Service      │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│Cache│ │ DB  │
└─────┘ └─────┘
```

### Components

1. **CDN:** Serve videos from edge locations
2. **Video Service:** Handle streaming logic
3. **Encoding Service:** Process uploaded videos
4. **Recommendation Service:** Suggest videos

---

## Video Processing Pipeline

### Encoding Flow

```
┌─────────────────────────────────────┐
│ 1. Upload video                     │
│    Content Creator → Upload Service │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 2. Store original video             │
│    - Save to object storage (S3)    │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 3. Encode to multiple qualities     │
│    - 4K (25 Mbps)                   │
│    - 1080p (8 Mbps)                 │
│    - 720p (4 Mbps)                  │
│    - 480p (2 Mbps)                  │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 4. Store encoded videos              │
│    - Upload to CDN                  │
│    - Store metadata in database     │
└─────────────────────────────────────┘
```

---

## CDN Architecture

### Geographic Distribution

```
Origin Server (US):
┌─────────────────┐
│  Video Storage   │
└─────────────────┘
         │
    ┌────┼────┐
    │    │    │
    ▼    ▼    ▼
┌────┐ ┌────┐ ┌────┐
│US  │ │EU  │ │Asia│
│Edge│ │Edge│ │Edge│
└────┘ └────┘ └────┘
```

### How CDN Works

```
User in India requests video:
1. Request goes to Asia edge server
2. If cached → Serve immediately
3. If not cached → Fetch from origin
4. Cache for future requests
```

---

## Adaptive Bitrate Streaming

### How It Works

```
Client monitors network:
- High bandwidth → Request 4K quality
- Medium bandwidth → Request 1080p
- Low bandwidth → Request 720p
- Very low → Request 480p

Video split into chunks:
- Each chunk in multiple qualities
- Client requests appropriate quality
- Seamless switching between qualities
```

### Benefits

- ✅ No buffering
- ✅ Best quality for network
- ✅ Smooth playback

---

## Database Design

### Schema

```sql
CREATE TABLE videos (
    id BIGINT PRIMARY KEY,
    title VARCHAR(255),
    description TEXT,
    duration INT,  -- seconds
    created_at TIMESTAMP,
    INDEX idx_created (created_at)
);

CREATE TABLE video_qualities (
    video_id BIGINT,
    quality VARCHAR(20),  -- 4K, 1080p, 720p, 480p
    bitrate INT,  -- Mbps
    cdn_url VARCHAR(500),
    PRIMARY KEY (video_id, quality)
);

CREATE TABLE watch_history (
    user_id BIGINT,
    video_id BIGINT,
    watched_seconds INT,
    last_watched_at TIMESTAMP,
    PRIMARY KEY (user_id, video_id)
);
```

---

## Scaling the System

### 1. CDN Distribution

```
Multiple CDN providers:
- CloudFlare
- AWS CloudFront
- Akamai

Geographic distribution:
- Serve from nearest edge
- Reduce latency
```

### 2. Video Chunking

```
Split videos into chunks:
- 10-second chunks
- Multiple qualities per chunk
- Parallel download
```

### 3. Predictive Caching

```
Cache popular videos:
- Top 100 videos in all edges
- Pre-warm cache
- Reduce origin load
```

---

## Summary

**Key Design Decisions:**
1. ✅ **CDN:** Geographic distribution
2. ✅ **Multiple Qualities:** Adaptive streaming
3. ✅ **Chunking:** Parallel download
4. ✅ **Predictive Caching:** Popular videos
5. ✅ **Encoding Pipeline:** Process uploads

**Next Steps:**
- Learn [Designing Search Engine](./17_DESIGNING_SEARCH_ENGINE.md)
- Understand [CDN Guide](../imp/CDN_GUIDE.md)
- Practice [More System Designs](./README.md)

---

**References:**
- [Netflix Architecture](https://netflix.com)
- [Video Streaming](https://algomaster.io/learn/system-design)

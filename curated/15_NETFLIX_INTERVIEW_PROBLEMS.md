# Netflix Interview Problems - Real Questions & Solutions

> **Curated from actual Netflix staff engineer interviews**

---

## System Design Problems

### Problem 1: Design Netflix Video Streaming

**Problem Statement:**
Design a video streaming service that can serve millions of concurrent users.

**Requirements:**
- 200M users
- 1B hours per day
- Low latency (< 2 seconds)
- Adaptive bitrate streaming

**Solution:**
- **CDN**: Edge locations for content delivery
- **Adaptive Bitrate**: HLS/DASH protocols
- **Transcoding**: Multiple quality versions
- **Pre-warming**: Cache popular content

---

### Problem 2: Design Netflix Recommendation System

**Problem Statement:**
Design a recommendation system that suggests movies and shows.

**Requirements:**
- Real-time recommendations
- Personalization
- Handle cold start
- Multiple algorithms

**Solution:**
- **Collaborative Filtering**: User-based, item-based
- **Content-Based**: Movie features
- **Deep Learning**: Neural networks
- **A/B Testing**: Algorithm selection

---

*This guide contains real Netflix interview problems covering video streaming, CDN, recommendation systems, and large-scale distributed systems.*

---

*This guide is continuously updated with new Netflix interview problems.*


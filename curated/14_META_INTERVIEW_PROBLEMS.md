# Meta/Facebook Interview Problems - Real Questions & Solutions

> **Curated from actual Meta staff engineer interviews**

---

## System Design Problems

### Problem 1: Design Facebook News Feed

**Problem Statement:**
Design a news feed system that shows posts from friends and pages.

**Requirements:**
- 2B users
- 1B posts per day
- Feed generation < 200ms
- Personalized ranking

**Solution:**
- **Hybrid Approach**: Push for normal users, pull for celebrities
- **Ranking**: ML model for relevance
- **Caching**: Pre-computed feeds
- **Real-time**: Event-driven updates

---

### Problem 2: Design Facebook Messenger

**Problem Statement:**
Design a messaging system supporting 1B+ users.

**Requirements:**
- Real-time messaging
- Message persistence
- Read receipts
- Group chats

**Solution:**
- **WebSockets**: Real-time communication
- **Message Queue**: Reliable delivery
- **Database**: Cassandra for messages
- **Caching**: Redis for online status

---

*This guide contains real Meta interview problems covering social networks, real-time systems, and large-scale distributed systems.*

---

*This guide is continuously updated with new Meta interview problems.*


# Designing a Chat System (WhatsApp)

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Requirements Gathering](#requirements-gathering)
3. [Capacity Estimation](#capacity-estimation)
4. [High-Level Design](#high-level-design)
5. [Detailed Design](#detailed-design)
6. [Message Delivery](#message-delivery)
7. [Scaling the System](#scaling-the-system)

---

## Problem Statement

Design a real-time chat system like WhatsApp that supports:
- One-on-one messaging
- Group messaging
- Message delivery status (sent, delivered, read)
- Presence (online/offline)
- Media sharing (images, videos)

---

## Requirements Gathering

### Functional Requirements

1. **Messaging**
   - Send/receive messages
   - Support text, images, videos
   - Message delivery status
   - Read receipts

2. **Presence**
   - Show online/offline status
   - Last seen timestamp

3. **Group Chat**
   - Create groups
   - Add/remove members
   - Group admin

### Non-Functional Requirements

1. **Scale**
   - 1 billion users
   - 50 billion messages/day
   - 1:1 message ratio (send/receive)

2. **Performance**
   - Message delivery: < 100ms
   - Real-time updates

3. **Availability**
   - 99.9% uptime
   - Handle network failures

---

## Capacity Estimation

### Traffic Estimates

**Messages:**
```
50B messages/day
= 50B / (24 × 60 × 60) seconds
= 578,704 messages/second
≈ 600K messages/second
```

**Users:**
```
1B users
Average: 50 messages/user/day
Peak: 100 messages/user/day
```

**Storage:**
```
Per message: 100 bytes (average)
50B messages/day × 100 bytes = 5TB/day
Retention: 1 year = 1.8PB
```

---

## High-Level Design

### Architecture

```
┌──────────┐
│  Client  │
│ (Mobile) │
└────┬─────┘
     │ WebSocket
     ▼
┌──────────────┐
│ Chat Service  │
│ (WebSocket)   │
└──────┬───────┘
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

1. **WebSocket Server:** Real-time communication
2. **Message Service:** Handle message logic
3. **Presence Service:** Track online/offline
4. **Database:** Store messages
5. **Cache:** Store active sessions

---

## Detailed Design

### 1. Message Sending Flow

```
┌─────────────────────────────────────┐
│ 1. User A sends message to User B   │
│    Client → WebSocket Server        │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 2. Validate message                 │
│    - Check if User B exists         │
│    - Check if blocked               │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 3. Store message in database        │
│    - Save to messages table        │
│    - Generate message ID            │
└──────────────┬──────────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
┌──────────┐   ┌──────────┐
│  Cache   │   │  Check   │
│ Message  │   │  Online  │
└──────────┘   └──────────┘
        │             │
        │             ▼
        │      ┌──────────┐
        │      │ User B   │
        │      │ Online?  │
        │      └────┬─────┘
        │           │
        │      ┌────┴────┐
        │      │         │
        │      ▼         ▼
        │  ┌──────┐  ┌──────┐
        │  │ Yes  │  │ No   │
        │  └──┬───┘  └──┬───┘
        │     │         │
        │     │         ▼
        │     │    ┌──────────┐
        │     │    │ Store in │
        │     │    │ Queue    │
        │     │    └──────────┘
        │     │
        │     ▼
        │ ┌──────────┐
        │ │ Send via │
        │ │ WebSocket│
        │ └──────────┘
        │
        ▼
┌─────────────────────────────────────┐
│ 4. Update delivery status            │
│    - Sent: Immediately              │
│    - Delivered: When User B receives│
│    - Read: When User B reads        │
└─────────────────────────────────────┘
```

### 2. Message Receiving Flow

```
┌─────────────────────────────────────┐
│ 1. User B receives message           │
│    WebSocket Server → Client        │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 2. Update delivery status           │
│    - Mark as delivered              │
│    - Update in database             │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 3. User B reads message             │
│    - Mark as read                   │
│    - Update read receipt            │
│    - Notify User A                  │
└─────────────────────────────────────┘
```

---

## Message Delivery

### Delivery Status

```
Sent: Message sent from User A
Delivered: Message received by User B's device
Read: User B opened the message
```

### Implementation

```java
public enum MessageStatus {
    SENT,
    DELIVERED,
    READ
}

public void sendMessage(Message message) {
    // 1. Store message
    messageRepository.save(message);
    
    // 2. Update status to SENT
    updateStatus(message.getId(), MessageStatus.SENT);
    
    // 3. Check if recipient is online
    if (presenceService.isOnline(message.getRecipientId())) {
        // 4. Send via WebSocket
        websocketService.send(message);
        updateStatus(message.getId(), MessageStatus.DELIVERED);
    } else {
        // 5. Store in queue for later
        messageQueue.enqueue(message);
    }
}
```

---

## Presence Management

### Online/Offline Status

```
User connects → Mark as online
User disconnects → Mark as offline
Heartbeat → Keep connection alive
```

### Implementation

```java
@OnOpen
public void onOpen(Session session, String userId) {
    presenceService.markOnline(userId);
    // Send pending messages
    sendPendingMessages(userId);
}

@OnClose
public void onClose(String userId) {
    presenceService.markOffline(userId);
    presenceService.updateLastSeen(userId);
}

@Scheduled(fixedRate = 30000)  // Every 30 seconds
public void heartbeat() {
    // Keep connections alive
    websocketService.sendHeartbeat();
}
```

---

## Database Design

### Schema

```sql
CREATE TABLE messages (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    sender_id BIGINT NOT NULL,
    recipient_id BIGINT,
    group_id BIGINT,
    content TEXT NOT NULL,
    message_type VARCHAR(20),  -- text, image, video
    status VARCHAR(20),  -- sent, delivered, read
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_recipient (recipient_id, created_at),
    INDEX idx_group (group_id, created_at)
);

CREATE TABLE groups (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255),
    admin_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE group_members (
    group_id BIGINT,
    user_id BIGINT,
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (group_id, user_id)
);

CREATE TABLE presence (
    user_id BIGINT PRIMARY KEY,
    status VARCHAR(20),  -- online, offline
    last_seen TIMESTAMP,
    INDEX idx_status (status)
);
```

---

## Scaling the System

### 1. WebSocket Connection Management

```
┌──────────────┐
│ Load         │
│ Balancer    │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│ WS  │ │ WS  │
│Srv 1│ │Srv 2│
└─────┘ └─────┘
```

**Challenge:** Sticky sessions needed (same user → same server)

---

### 2. Message Queue for Offline Users

```
User offline:
Message → Queue → When online → Deliver
```

---

### 3. Database Sharding

```
Shard by user_id:
hash(user_id) % number_of_shards = shard_number
```

---

## Summary

**Key Design Decisions:**
1. ✅ **WebSocket:** Real-time communication
2. ✅ **Message Queue:** Offline message delivery
3. ✅ **Presence Service:** Track online/offline
4. ✅ **Database Sharding:** Scale storage
5. ✅ **Caching:** Active sessions, recent messages

**Next Steps:**
- Learn [Designing News Feed](./15_DESIGNING_NEWS_FEED.md)
- Understand [Event-Driven Architecture](./08_EVENT_DRIVEN_ARCHITECTURE.md)
- Practice [More System Designs](./README.md)

---

**References:**
- [WhatsApp Architecture](https://www.whatsapp.com)
- [WebSocket Documentation](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)

# Distributed Systems

## Table of Contents
1. [Introduction](#introduction)
2. [Consensus Algorithms](#consensus-algorithms)
3. [Distributed Locks](#distributed-locks)
4. [Leader Election](#leader-election)
5. [Distributed Transactions](#distributed-transactions)
6. [Vector Clocks](#vector-clocks)
7. [Real-World Examples](#real-world-examples)

---

## Introduction

**Distributed Systems** are systems where components are on different networked computers that communicate and coordinate by passing messages.

### Challenges

1. **Network Failures:** Messages can be lost or delayed
2. **Partial Failures:** Some nodes fail, others continue
3. **Clock Synchronization:** No global clock
4. **Consistency:** Maintaining data consistency

---

## Consensus Algorithms

**Consensus** is the process of agreeing on a value among distributed nodes.

### Raft Algorithm

**Simpler than Paxos, easier to understand.**

**Key Concepts:**
- **Leader:** One node handles all client requests
- **Followers:** Replicate leader's log
- **Candidate:** Node seeking to become leader

**States:**
```
Follower → Candidate → Leader
```

**Leader Election:**
```
1. Follower doesn't hear from leader (timeout)
2. Becomes candidate
3. Requests votes from other nodes
4. If majority votes → Becomes leader
```

**Example:**
```
5 nodes: A, B, C, D, E
Leader: A

A fails:
B, C, D, E become candidates
B gets votes from C, D → B becomes leader
```

---

### Paxos Algorithm

**More complex, handles Byzantine failures.**

**Phases:**
1. **Prepare:** Proposer sends proposal number
2. **Promise:** Acceptors promise not to accept lower numbers
3. **Accept:** Proposer sends value, acceptors accept

**Use Case:** Google Chubby, Apache Zookeeper

---

## Distributed Locks

**Prevent concurrent access to shared resources.**

### Redis Distributed Lock

**Implementation:**
```java
public boolean acquireLock(String lockKey, int timeoutSeconds) {
    String lockValue = UUID.randomUUID().toString();
    
    // Try to acquire lock
    Boolean acquired = redisTemplate.opsForValue()
        .setIfAbsent(lockKey, lockValue, Duration.ofSeconds(timeoutSeconds));
    
    if (acquired) {
        // Lock acquired
        return true;
    }
    return false;
}

public void releaseLock(String lockKey, String lockValue) {
    // Only release if we own the lock
    String currentValue = redisTemplate.opsForValue().get(lockKey);
    if (lockValue.equals(currentValue)) {
        redisTemplate.delete(lockKey);
    }
}
```

### Challenges

1. **Deadlock:** Lock holder crashes
   - **Solution:** Set expiration time

2. **Lock Renewal:** Long operations
   - **Solution:** Renew lock periodically

---

## Leader Election

**Select one node as leader in a distributed system.**

### Approaches

#### 1. **Bully Algorithm**

**Highest ID wins:**
```
Nodes: A(1), B(2), C(3)
Leader: C (highest ID)

C fails:
A and B detect
B sends "I'm leader" to A
A accepts (B has higher ID)
B becomes leader
```

---

#### 2. **Ring Algorithm**

**Election message passes around ring:**
```
Nodes: A → B → C → A (ring)

A detects leader failure:
A sends election message to B
B forwards to C
C becomes leader (highest ID)
```

---

#### 3. **Zookeeper Leader Election**

**Use Zookeeper ephemeral nodes:**
```
1. Create ephemeral node with sequence
2. Node with lowest sequence becomes leader
3. If leader fails, next node becomes leader
```

---

## Distributed Transactions

### Two-Phase Commit (2PC)

**Coordinator manages transaction across nodes.**

**Phase 1: Prepare**
```
Coordinator → All participants: "Prepare to commit"
Participants → Prepare and respond "Yes" or "No"
```

**Phase 2: Commit/Abort**
```
If all say "Yes":
Coordinator → All: "Commit"
Else:
Coordinator → All: "Abort"
```

**Problem:** Blocking (waits for all participants)

---

### Saga Pattern

**Distributed transactions without locks.**

**Types:**

**1. Choreography:**
```
Service A → Event → Service B
Service B → Event → Service C
If C fails → C sends compensation event
```

**2. Orchestration:**
```
Orchestrator coordinates:
1. Call Service A
2. Call Service B
3. Call Service C
If C fails → Call compensation for A and B
```

---

## Vector Clocks

**Track causality in distributed systems.**

### Problem

**Without vector clocks:**
```
Event A happens before Event B
But timestamps: A=10:00:01, B=10:00:00
Can't determine order ❌
```

### Solution

**Vector Clock:**
```
Each node maintains vector:
Node A: [A:2, B:1, C:0]
Node B: [A:1, B:3, C:0]
Node C: [A:1, B:1, C:2]

Compare vectors to determine causality
```

---

## Real-World Examples

### Example 1: Apache Zookeeper

**Uses:**
- Configuration management
- Leader election
- Distributed locks
- Service discovery

**Consensus:** ZAB (Zookeeper Atomic Broadcast)

---

### Example 2: etcd

**Uses:**
- Kubernetes configuration
- Service discovery
- Distributed coordination

**Consensus:** Raft

---

### Example 3: Google Chubby

**Uses:**
- Lock service
- Configuration storage
- Leader election

**Consensus:** Paxos

---

## Summary

**Key Concepts:**
1. ✅ **Consensus:** Raft, Paxos
2. ✅ **Distributed Locks:** Redis, Zookeeper
3. ✅ **Leader Election:** Bully, Ring, Zookeeper
4. ✅ **Transactions:** 2PC, Saga
5. ✅ **Vector Clocks:** Causality tracking

**Next Steps:**
- Learn [System Monitoring](./22_SYSTEM_MONITORING.md)
- Understand [Security](./23_SECURITY_IN_SYSTEM_DESIGN.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [Distributed Systems](https://algomaster.io/learn/system-design)
- [Raft Algorithm](https://raft.github.io/)

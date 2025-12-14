# Distributed Systems Problems - 20+ Real-World Problems

> **Curated from top tech companies and real production scenarios**

This guide contains 20+ real-world distributed systems problems with comprehensive solutions, suitable for staff engineer interviews.

---

## Table of Contents

1. [Consistency Problems](#consistency-problems)
2. [Availability Problems](#availability-problems)
3. [Partition Tolerance Problems](#partition-tolerance-problems)
4. [Consensus Problems](#consensus-problems)
5. [Distributed Lock Problems](#distributed-lock-problems)

---

## Consistency Problems

### Problem 1: Design Eventually Consistent System

**Problem Statement:**
Design a system that can handle eventual consistency across multiple data centers.

**Requirements:**
- Multiple data centers
- Eventual consistency
- Conflict resolution
- High availability

**Solution:**

```java
// Vector Clocks for Event Ordering
public class VectorClock {
    private Map<String, Long> clock = new HashMap<>();
    
    public VectorClock tick(String nodeId) {
        clock.put(nodeId, clock.getOrDefault(nodeId, 0L) + 1);
        return this;
    }
    
    public VectorClock update(VectorClock other) {
        for (Map.Entry<String, Long> entry : other.clock.entrySet()) {
            clock.put(entry.getKey(), 
                Math.max(clock.getOrDefault(entry.getKey(), 0L), entry.getValue()));
        }
        return this;
    }
    
    public boolean happensBefore(VectorClock other) {
        boolean lessThan = false;
        for (Map.Entry<String, Long> entry : clock.entrySet()) {
            Long otherValue = other.clock.getOrDefault(entry.getKey(), 0L);
            if (entry.getValue() > otherValue) {
                return false;
            }
            if (entry.getValue() < otherValue) {
                lessThan = true;
            }
        }
        return lessThan;
    }
}

// Conflict Resolution: Last Write Wins
public class ConflictResolver {
    public Data resolveConflict(Data data1, Data data2) {
        if (data1.getTimestamp() > data2.getTimestamp()) {
            return data1; // Last write wins
        }
        return data2;
    }
}

// Eventual Consistency with Replication
@Service
public class ReplicationService {
    @Autowired
    private List<DataCenter> dataCenters;
    
    public void write(String key, String value) {
        // Write to primary
        DataCenter primary = getPrimary();
        primary.write(key, value);
        
        // Replicate to secondaries (async)
        dataCenters.stream()
            .filter(dc -> dc != primary)
            .forEach(dc -> replicateAsync(dc, key, value));
    }
    
    public String read(String key) {
        // Read from any data center
        DataCenter dc = getNearestDataCenter();
        return dc.read(key);
    }
}
```

---

## Availability Problems

### Problem 2: Design High Availability System

**Problem Statement:**
Design a system with 99.99% availability (52 minutes downtime per year).

**Requirements:**
- Redundancy
- Failover
- Health checks
- Load balancing

**Solution:**

```java
// Health Check Service
@Component
public class HealthCheckService {
    @Scheduled(fixedRate = 5000) // Every 5 seconds
    public void checkHealth() {
        List<Server> servers = getServers();
        servers.parallelStream().forEach(server -> {
            boolean healthy = checkServerHealth(server);
            server.setHealthy(healthy);
            
            if (!healthy) {
                handleUnhealthyServer(server);
            }
        });
    }
    
    private boolean checkServerHealth(Server server) {
        try {
            ResponseEntity<String> response = restTemplate.getForEntity(
                server.getUrl() + "/health", String.class);
            return response.getStatusCode() == HttpStatus.OK;
        } catch (Exception e) {
            return false;
        }
    }
}

// Failover Mechanism
@Service
public class FailoverService {
    @Autowired
    private List<Server> servers;
    
    public Server getAvailableServer() {
        return servers.stream()
            .filter(Server::isHealthy)
            .findFirst()
            .orElseThrow(() -> new RuntimeException("No healthy servers"));
    }
    
    public void handleServerFailure(Server server) {
        server.setHealthy(false);
        // Remove from load balancer
        loadBalancer.removeServer(server);
        
        // Notify monitoring
        monitoringService.alert("Server " + server.getId() + " is down");
    }
}
```

---

## Consensus Problems

### Problem 3: Implement Raft Consensus Algorithm

**Problem Statement:**
Implement Raft consensus algorithm for distributed system coordination.

**Requirements:**
- Leader election
- Log replication
- Fault tolerance

**Solution:**

```java
// Raft Node States
public enum RaftState {
    FOLLOWER, CANDIDATE, LEADER
}

// Raft Node
public class RaftNode {
    private String nodeId;
    private RaftState state;
    private String currentLeader;
    private long currentTerm;
    private long lastHeartbeat;
    
    public void startElection() {
        state = RaftState.CANDIDATE;
        currentTerm++;
        
        // Request votes from other nodes
        int votes = requestVotes();
        
        if (votes > getTotalNodes() / 2) {
            becomeLeader();
        } else {
            state = RaftState.FOLLOWER;
        }
    }
    
    public void becomeLeader() {
        state = RaftState.LEADER;
        currentLeader = nodeId;
        
        // Send heartbeats to followers
        scheduleHeartbeats();
    }
    
    public void receiveHeartbeat(String leaderId, long term) {
        if (term > currentTerm) {
            currentTerm = term;
            currentLeader = leaderId;
            state = RaftState.FOLLOWER;
        }
        lastHeartbeat = System.currentTimeMillis();
    }
}
```

---

## Summary

These problems cover:
- ✅ Consistency patterns
- ✅ Availability patterns
- ✅ Consensus algorithms
- ✅ Distributed locks
- ✅ CAP theorem trade-offs

**Key Concepts:**
- Eventual consistency
- High availability
- Consensus algorithms (Raft, Paxos)
- Distributed coordination
- Fault tolerance

---

*This guide is continuously updated with new problems and solutions.*


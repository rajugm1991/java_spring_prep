# Key Components of a System

## Table of Contents
1. [Load Balancers](#load-balancers)
2. [Databases](#databases)
3. [Caching](#caching)
4. [Message Queues](#message-queues)
5. [CDN (Content Delivery Network)](#cdn-content-delivery-network)
6. [API Gateway](#api-gateway)
7. [Service Discovery](#service-discovery)
8. [Monitoring & Logging](#monitoring--logging)

---

## Load Balancers

**Load Balancer** distributes incoming network traffic across multiple servers to ensure no single server is overwhelmed.

### Types of Load Balancers

#### 1. **Layer 4 (Transport Layer) - TCP/UDP**

Routes traffic based on IP address and port number.

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     ▼
┌──────────────┐
│ Layer 4 LB  │  ← Routes by IP:Port
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│ S1  │ │ S2  │
└─────┘ └─────┘
```

**Use Cases:**
- Simple routing
- High performance
- SSL termination

**Examples:** AWS Network Load Balancer, HAProxy

---

#### 2. **Layer 7 (Application Layer) - HTTP/HTTPS**

Routes traffic based on URL, headers, cookies, or content.

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     ▼
┌──────────────┐
│ Layer 7 LB  │  ← Routes by URL/Headers
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│ API │ │ Web │
└─────┘ └─────┘
```

**Use Cases:**
- Content-based routing
- SSL termination
- Request/response manipulation

**Examples:** AWS Application Load Balancer, NGINX

---

### Load Balancing Algorithms

#### 1. **Round Robin**

Distributes requests sequentially.

```
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
...
```

**Pros:** Simple, fair distribution
**Cons:** Doesn't consider server load

---

#### 2. **Least Connections**

Routes to server with fewest active connections.

```
Server 1: 10 connections
Server 2: 5 connections
Server 3: 8 connections

New Request → Server 2 (least connections)
```

**Pros:** Better for long-lived connections
**Cons:** Requires tracking connections

---

#### 3. **Weighted Round Robin**

Distributes based on server capacity.

```
Server 1 (Weight: 3) → Gets 3 requests
Server 2 (Weight: 2) → Gets 2 requests
Server 3 (Weight: 1) → Gets 1 request
```

**Pros:** Handles different server capacities
**Cons:** Requires weight configuration

---

#### 4. **IP Hash**

Routes based on client IP hash.

```
hash(client_ip) % number_of_servers = server_number

Same client IP → Always same server
```

**Pros:** Session affinity
**Cons:** Uneven distribution if IPs are clustered

---

### Real-World Example: E-Commerce Platform

```
┌─────────────────────────────────┐
│      Users (Worldwide)           │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│    Global Load Balancer         │
│    (Geographic Routing)         │
└──────────────┬──────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
┌──────────┐   ┌──────────┐
│ US Region│   │ EU Region│
│   LB     │   │   LB     │
└────┬─────┘   └────┬─────┘
     │             │
┌────┼────┐   ┌────┼────┐
│    │    │   │    │    │
▼    ▼    ▼   ▼    ▼    ▼
┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐
│S1│ │S2│ │S3│ │S4│ │S5│ │S6│
└──┘ └──┘ └──┘ └──┘ └──┘ └──┘
```

---

## Databases

**Database** stores and manages data persistently.

### Types of Databases

#### 1. **Relational Databases (SQL)**

Store data in tables with relationships.

**Examples:** PostgreSQL, MySQL, Oracle

**Use Cases:**
- Transactional data
- ACID compliance needed
- Complex queries with joins

**Example Schema:**
```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    total DECIMAL(10,2),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

#### 2. **NoSQL Databases**

Store data in flexible formats.

**Types:**

**Document Store (MongoDB):**
```json
{
  "user_id": 123,
  "name": "John",
  "orders": [
    {"order_id": 1, "total": 100},
    {"order_id": 2, "total": 200}
  ]
}
```

**Key-Value Store (Redis):**
```
key: "user:123"
value: {"name": "John", "email": "john@example.com"}
```

**Column Store (Cassandra):**
```
Row Key: user_123
Columns: name, email, age
```

**Graph Database (Neo4j):**
```
Nodes: User, Product
Relationships: PURCHASED, LIKES
```

---

### Database Selection Guide

| Database Type | Use Case | Example |
|--------------|----------|---------|
| **PostgreSQL** | Complex queries, ACID | E-commerce, Banking |
| **MySQL** | Web applications | WordPress, CMS |
| **MongoDB** | Flexible schema | Content management |
| **Redis** | Caching, Sessions | Session store, Cache |
| **Cassandra** | High write throughput | IoT, Analytics |
| **Neo4j** | Graph relationships | Social networks |

---

### Database Scaling Strategies

#### 1. **Read Replicas**

```
┌──────────┐
│  Master  │  ← Writes
└────┬─────┘
     │ Replication
     │
┌────┼────┬────┐
│    │    │    │
▼    ▼    ▼    ▼
┌──┐ ┌──┐ ┌──┐ ┌──┐
│R1│ │R2│ │R3│ │R4│  ← Reads
└──┘ └──┘ └──┘ └──┘
```

**Benefits:**
- Scale reads horizontally
- Reduce master load
- Geographic distribution

---

#### 2. **Sharding**

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Shard 1  │  │ Shard 2  │  │ Shard 3  │
│ (User    │  │ (User    │  │ (User    │
│  1-33M)  │  │  34-66M) │  │  67-100M)│
└──────────┘  └──────────┘  └──────────┘
```

**Sharding Strategies:**
- **Range-based:** User ID 1-1M → Shard 1
- **Hash-based:** hash(user_id) % 3 → Shard number
- **Directory-based:** Lookup table

---

## Caching

**Cache** stores frequently accessed data in fast storage.

### Cache Levels

#### 1. **Application Cache (In-Memory)**

```
┌─────────────────┐
│  Application    │
│  ┌───────────┐  │
│  │  Cache    │  │  ← In-process cache
│  │  (Local)  │  │
│  └───────────┘  │
└─────────────────┘
```

**Pros:** Fastest (nanoseconds)
**Cons:** Limited to single process

---

#### 2. **Distributed Cache (Redis/Memcached)**

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ App 1    │  │ App 2    │  │ App 3    │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │            │            │
     └────────────┼────────────┘
                  │
                  ▼
          ┌──────────┐
          │  Redis   │
          │  Cluster │
          └──────────┘
```

**Pros:** Shared across services
**Cons:** Network latency (milliseconds)

---

#### 3. **CDN Cache**

```
┌─────────────────┐
│  Origin Server  │
└────────┬─────────┘
         │
    ┌────┼────┐
    │    │    │
    ▼    ▼    ▼
┌────┐ ┌────┐ ┌────┐
│Edge│ │Edge│ │Edge│
│US  │ │EU  │ │Asia│
└────┘ └────┘ └────┘
```

**Pros:** Geographic distribution
**Cons:** Cache invalidation complexity

---

### Cache Strategies

#### 1. **Cache-Aside (Lazy Loading)**

```
Read:
1. Check cache
2. If miss → Read from DB
3. Update cache
4. Return data

Write:
1. Write to DB
2. Invalidate cache
```

**Example:**
```java
public Product getProduct(String id) {
    // Check cache
    Product product = cache.get("product:" + id);
    if (product != null) {
        return product;  // Cache hit
    }
    
    // Cache miss - read from DB
    product = database.getProduct(id);
    
    // Update cache
    cache.set("product:" + id, product, 3600);
    
    return product;
}
```

---

#### 2. **Write-Through**

```
Write:
1. Write to cache
2. Write to DB
3. Return

Read:
1. Read from cache (always available)
```

---

#### 3. **Write-Back (Write-Behind)**

```
Write:
1. Write to cache
2. Return immediately
3. Async write to DB (later)
```

---

## Message Queues

**Message Queue** enables asynchronous communication between services.

### Message Queue Benefits

1. **Decoupling:** Services don't need to know about each other
2. **Reliability:** Messages are persisted
3. **Scalability:** Handle traffic spikes
4. **Asynchronous:** Non-blocking operations

---

### Message Queue Types

#### 1. **Point-to-Point (Queue)**

```
Producer → Queue → Consumer
         (One message consumed by one consumer)
```

**Example:** Task queue, job processing

---

#### 2. **Pub-Sub (Topic)**

```
Producer → Topic → Multiple Consumers
         (One message consumed by multiple consumers)
```

**Example:** Event notifications, logging

---

### Popular Message Queues

#### 1. **Kafka**

**Use Cases:**
- Event streaming
- High throughput
- Event replay

**Architecture:**
```
Producer → Topic (Partitions) → Consumer Groups
```

**Example:**
```java
// Producer
kafkaTemplate.send("order-created", orderId, event);

// Consumer
@KafkaListener(topics = "order-created", groupId = "inventory-service")
public void handleOrderCreated(OrderCreatedEvent event) {
    inventoryService.reserveInventory(event);
}
```

---

#### 2. **RabbitMQ**

**Use Cases:**
- Task queues
- RPC
- Message routing

**Architecture:**
```
Producer → Exchange → Queue → Consumer
```

---

#### 3. **Amazon SQS**

**Use Cases:**
- Cloud-based queues
- Simple integration
- Managed service

---

### Real-World Example: Order Processing

```
┌──────────┐
│  Order   │
│ Service  │
└────┬─────┘
     │ Publishes event
     ▼
┌──────────┐
│  Kafka   │
│  Topic   │
└────┬─────┘
     │
┌────┼────┬────┐
│    │    │    │
▼    ▼    ▼    ▼
┌──┐ ┌──┐ ┌──┐ ┌──┐
│I1│ │P1│ │N1│ │A1│
│nv│ │ay│ │ot│ │na│
│en│ │me│ │if│ │ly│
│to│ │nt│ │  │ │ti│
│ry│ │  │ │  │ │cs│
└──┘ └──┘ └──┘ └──┘
```

---

## CDN (Content Delivery Network)

**CDN** delivers content from geographically distributed servers.

### How CDN Works

```
Without CDN:
User (India) → Origin Server (US) → Response (500ms) ❌

With CDN:
User (India) → CDN Edge (India) → Response (50ms) ✅
```

---

### CDN Architecture

```
┌─────────────────────────────────┐
│      Origin Server              │
│      (US - Single Location)     │
└─────────────────────────────────┘
              │
    ┌─────────┼─────────┐
    │         │         │
    ▼         ▼         ▼
┌──────┐ ┌──────┐ ┌──────┐
│India │ │Europe│ │Asia  │
│Edge  │ │Edge  │ │Edge  │
└──────┘ └──────┘ └──────┘
```

---

### CDN Use Cases

1. **Static Assets:** Images, CSS, JavaScript
2. **Video Streaming:** Netflix, YouTube
3. **API Responses:** Cached API responses
4. **Software Downloads:** Large files

---

### CDN Providers

- **CloudFlare:** DDoS protection, global network
- **AWS CloudFront:** Integrated with AWS
- **Akamai:** Enterprise-grade CDN
- **Fastly:** Real-time cache invalidation

---

## API Gateway

**API Gateway** is a single entry point for all client requests.

### API Gateway Functions

1. **Routing:** Route requests to appropriate services
2. **Authentication:** Verify JWT tokens
3. **Rate Limiting:** Prevent abuse
4. **Load Balancing:** Distribute traffic
5. **Request/Response Transformation:** Modify data
6. **Monitoring:** Log requests, metrics

---

### API Gateway Architecture

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     ▼
┌──────────────┐
│ API Gateway  │
│ - Auth       │
│ - Rate Limit │
│ - Routing    │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│User │ │Order│
│Svc  │ │Svc  │
└─────┘ └─────┘
```

---

### API Gateway Example: Spring Cloud Gateway

```java
@Configuration
public class GatewayConfig {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("user-service", r -> r
                .path("/api/users/**")
                .uri("lb://user-service"))
            .route("order-service", r -> r
                .path("/api/orders/**")
                .uri("lb://order-service"))
            .build();
    }
}
```

---

## Service Discovery

**Service Discovery** helps services find and communicate with each other.

### Service Discovery Patterns

#### 1. **Client-Side Discovery**

```
Client → Service Registry → Get Service List
Client → Directly connect to Service
```

**Example:** Eureka (Netflix)

---

#### 2. **Server-Side Discovery**

```
Client → Load Balancer → Service Registry → Service
```

**Example:** AWS ELB, Kubernetes

---

### Service Registry Example: Eureka

```
┌──────────┐
│ Service  │
│   A      │
└────┬─────┘
     │ Registers
     ▼
┌──────────────┐
│   Eureka     │
│   Registry   │
└──────┬───────┘
       │
       │ Discovers
       ▼
┌──────────┐
│ Service  │
│   B      │
└──────────┘
```

---

## Monitoring & Logging

### Three Pillars of Observability

#### 1. **Logs**

**What happened?**
- Application logs
- Error logs
- Access logs

**Example:** ELK Stack (Elasticsearch, Logstash, Kibana)

---

#### 2. **Metrics**

**How is it performing?**
- CPU usage
- Memory usage
- Request rate
- Error rate

**Example:** Prometheus + Grafana

---

#### 3. **Traces**

**Where is the bottleneck?**
- Request flow
- Service dependencies
- Latency breakdown

**Example:** Jaeger, Zipkin

---

### Monitoring Stack

```
┌─────────────────────────────────┐
│      Application                │
└──────────┬──────────────────────┘
           │
    ┌──────┼──────┐
    │      │      │
    ▼      ▼      ▼
┌──────┐ ┌──────┐ ┌──────┐
│ Logs  │ │Metrics│ │Traces│
│(ELK)  │ │(Prom) │ │(Jaeger)│
└──────┘ └──────┘ └──────┘
```

---

## Summary

**Key Components:**
1. ✅ **Load Balancers:** Distribute traffic
2. ✅ **Databases:** Store data (SQL/NoSQL)
3. ✅ **Caching:** Speed up access
4. ✅ **Message Queues:** Async communication
5. ✅ **CDN:** Geographic content delivery
6. ✅ **API Gateway:** Single entry point
7. ✅ **Service Discovery:** Find services
8. ✅ **Monitoring:** Observe system health

**Next Steps:**
- Learn [Scalability Patterns](./04_SCALABILITY_PATTERNS.md)
- Understand [Database Design](./10_DATABASE_DESIGN.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [System Design Components](https://algomaster.io/learn/system-design)
- [Architecture Patterns](https://www.geeksforgeeks.org/system-design/)


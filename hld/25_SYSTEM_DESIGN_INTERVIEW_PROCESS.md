# System Design Interview Process

## Table of Contents
1. [Interview Structure](#interview-structure)
2. [Step-by-Step Approach](#step-by-step-approach)
3. [Requirements Clarification](#requirements-clarification)
4. [Capacity Estimation](#capacity-estimation)
5. [High-Level Design](#high-level-design)
6. [Detailed Design](#detailed-design)
7. [Common Mistakes](#common-mistakes)
8. [Communication Tips](#communication-tips)

---

## Interview Structure

### Typical Format

**Duration:** 45-60 minutes

**Phases:**
1. **Requirements (5-10 min):** Clarify functional and non-functional requirements
2. **Estimation (5 min):** Back-of-the-envelope calculations
3. **High-Level Design (10-15 min):** Core components and architecture
4. **Detailed Design (15-20 min):** Deep dive into components
5. **Discussion (5-10 min):** Trade-offs, optimizations, Q&A

---

## Step-by-Step Approach

### Step 1: Understand the Problem

**Ask clarifying questions:**

1. **Scale:**
   - How many users?
   - How many requests per second?
   - Expected growth?

2. **Features:**
   - What are the core features?
   - What are nice-to-have features?
   - Any constraints?

3. **Performance:**
   - What's acceptable latency?
   - What's the read/write ratio?

4. **Consistency:**
   - Strong or eventual consistency?
   - What data is critical?

---

### Step 2: Capacity Estimation

**Estimate:**
- Traffic (QPS)
- Storage
- Bandwidth
- Memory

**Example:**
```
Users: 100M
Daily Active Users: 10M
Requests per user per day: 100
Total requests/day: 1B
QPS: 1B / (24 × 60 × 60) = 11,574 ≈ 12K QPS
```

---

### Step 3: High-Level Design

**Sketch core components:**

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     ▼
┌──────────────┐
│ Load         │
│ Balancer    │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│ API │ │ API │
└─────┘ └─────┘
```

---

### Step 4: Detailed Design

**Deep dive into:**
- Database schema
- API design
- Caching strategy
- Scaling approach

---

### Step 5: Identify Bottlenecks

**Consider:**
- Single points of failure
- Scalability limits
- Performance bottlenecks
- Data consistency issues

---

## Requirements Clarification

### Functional Requirements

**Ask:**
- What are the core features?
- What are the edge cases?
- What are the constraints?

**Example: URL Shortener**
- Shorten long URLs
- Redirect to original URL
- Track analytics
- URL expiration?

---

### Non-Functional Requirements

**Ask:**
- Scale (users, QPS)
- Performance (latency)
- Availability (uptime)
- Consistency (strong/eventual)

**Example:**
- 100M URLs/day
- 200ms redirect latency
- 99.9% availability
- Eventual consistency acceptable

---

## Capacity Estimation

### Traffic Estimation

```
Daily Active Users: 10M
Requests per user per day: 100
Total requests/day: 1B

QPS = 1B / (24 × 60 × 60) = 11,574
Peak QPS = 11,574 × 3 = 34,722 ≈ 35K QPS
```

### Storage Estimation

```
Per URL: 500 bytes
100M URLs/day × 365 days = 36.5B URLs/year
36.5B × 500 bytes = 18.25 TB/year
```

### Bandwidth Estimation

```
Read:write ratio = 100:1
Read QPS = 35K × 100 = 3.5M QPS
3.5M × 500 bytes = 1.75 GB/second
```

---

## High-Level Design

### Core Components

1. **Load Balancer:** Distribute traffic
2. **API Servers:** Handle requests
3. **Database:** Store data
4. **Cache:** Speed up access
5. **CDN:** Serve static content

### Architecture Diagram

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     ▼
┌──────────────┐
│ Load         │
│ Balancer    │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│ API │ │ API │
└──┬──┘ └──┬──┘
   │       │
   └───┬───┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│Cache│ │ DB  │
└─────┘ └─────┘
```

---

## Detailed Design

### Database Schema

```sql
CREATE TABLE urls (
    id BIGINT PRIMARY KEY,
    short_url VARCHAR(7) UNIQUE,
    long_url VARCHAR(2048),
    created_at TIMESTAMP,
    expires_at TIMESTAMP
);
```

### API Design

```
POST /api/v1/shorten
GET  /{short_url}
GET  /api/v1/analytics/{short_url}
```

### Caching Strategy

```
Cache 20% most popular URLs
Cache hit rate: 80%
Reduce database load by 80%
```

---

## Common Mistakes

### 1. **Jumping to Solutions**

**Bad:**
"I'll use Redis for caching"

**Good:**
"Let me first understand the requirements, then propose caching"

---

### 2. **Ignoring Scale**

**Bad:**
"Single database is fine"

**Good:**
"With 100M URLs/day, we need database sharding"

---

### 3. **Over-Engineering**

**Bad:**
"Let's use microservices, Kafka, Redis, MongoDB, Cassandra..."

**Good:**
"Start simple, add complexity only when needed"

---

### 4. **Not Discussing Trade-offs**

**Bad:**
"This is the best solution"

**Good:**
"This approach has pros and cons. Let me explain..."

---

### 5. **Not Asking Questions**

**Bad:**
Assumes requirements

**Good:**
Asks clarifying questions first

---

## Communication Tips

### 1. **Think Out Loud**

Explain your thought process:
"I'm considering two approaches. Let me think through the trade-offs..."

---

### 2. **Draw Diagrams**

Visualize your design:
"Let me draw the architecture..."

---

### 3. **Start Simple**

Begin with basic design, then add complexity:
"Let's start with a simple design, then optimize..."

---

### 4. **Discuss Trade-offs**

Explain pros and cons:
"This approach is faster but uses more memory..."

---

### 5. **Be Open to Feedback**

Accept suggestions:
"That's a good point. Let me reconsider..."

---

## Interview Checklist

### Before Interview

- [ ] Review common system design problems
- [ ] Practice back-of-the-envelope calculations
- [ ] Review scalability patterns
- [ ] Prepare questions to ask

### During Interview

- [ ] Clarify requirements first
- [ ] Estimate capacity
- [ ] Design high-level architecture
- [ ] Deep dive into components
- [ ] Discuss trade-offs
- [ ] Identify bottlenecks

### After Interview

- [ ] Reflect on what went well
- [ ] Identify areas for improvement
- [ ] Review your design
- [ ] Learn from feedback

---

## Summary

**Key Steps:**
1. ✅ **Clarify Requirements:** Ask questions
2. ✅ **Estimate Capacity:** Calculate scale
3. ✅ **High-Level Design:** Core components
4. ✅ **Detailed Design:** Deep dive
5. ✅ **Discuss Trade-offs:** Pros and cons
6. ✅ **Communicate Clearly:** Think out loud

**Next Steps:**
- Practice [Back-of-the-Envelope Calculations](./26_BACK_OF_THE_ENVELOPE.md)
- Review [Common Interview Questions](./27_COMMON_INTERVIEW_QUESTIONS.md)
- Study [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [System Design Interview Guide](https://algomaster.io/learn/system-design)
- [Interview Best Practices](https://www.geeksforgeeks.org/system-design/)

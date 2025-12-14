# Back-of-the-Envelope Calculations

## Table of Contents
1. [Introduction](#introduction)
2. [Power of 2](#power-of-2)
3. [Latency Numbers](#latency-numbers)
4. [Capacity Planning](#capacity-planning)
5. [Storage Calculations](#storage-calculations)
6. [Bandwidth Calculations](#bandwidth-calculations)
7. [Practice Problems](#practice-problems)

---

## Introduction

**Back-of-the-Envelope Calculations** are rough estimates used to:
- Understand system scale
- Make design decisions
- Identify bottlenecks
- Plan capacity

**Key Principle:** Round numbers, use approximations

---

## Power of 2

**Memorize these:**

```
2^10 = 1,024 ≈ 1,000 (1K)
2^20 = 1,048,576 ≈ 1,000,000 (1M)
2^30 = 1,073,741,824 ≈ 1,000,000,000 (1B)
2^40 = 1,099,511,627,776 ≈ 1,000,000,000,000 (1T)
```

**Useful for:**
- Memory sizes
- Storage calculations
- Network capacity

---

## Latency Numbers

**Memorize these (approximate):**

```
L1 cache reference:           1 ns
L2 cache reference:           4 ns
Main memory reference:       100 ns
SSD random read:          10,000 ns (10 μs)
Network round trip (same DC): 500,000 ns (0.5 ms)
Disk seek:               10,000,000 ns (10 ms)
```

**Key Insight:**
- Memory is 100x faster than disk
- Network is 5x faster than disk
- Cache is 100x faster than memory

---

## Capacity Planning

### Step 1: Understand Scale

**Questions to ask:**
- How many users?
- How many requests per user?
- What's the peak traffic?

**Example:**
```
Total users: 100M
Daily active users: 10M (10%)
Requests per user per day: 100
Total requests/day: 10M × 100 = 1B requests/day
```

---

### Step 2: Calculate QPS

**Queries Per Second:**

```
QPS = Total requests / (24 × 60 × 60)

Example:
1B requests/day / 86,400 seconds = 11,574 QPS
Peak QPS = Average QPS × 3 = 34,722 QPS ≈ 35K QPS
```

---

### Step 3: Estimate Storage

**Per Record Size:**

```
Example: URL Shortener
- Short URL: 7 bytes
- Long URL: 500 bytes (average)
- Timestamp: 7 bytes
- Click count: 4 bytes
Total: 518 bytes ≈ 500 bytes per URL
```

**Total Storage:**

```
100M URLs/day × 365 days = 36.5B URLs
36.5B × 500 bytes = 18.25 TB/year
With indexes: 18.25 × 1.5 = 27.4 TB/year ≈ 30 TB/year
```

---

### Step 4: Estimate Bandwidth

**Read/Write Ratio:**

```
Write QPS: 1,200
Read QPS: 120,000 (100:1 ratio)

Write bandwidth: 1,200 × 500 bytes = 600 KB/s
Read bandwidth: 120,000 × 500 bytes = 60 MB/s
```

---

## Storage Calculations

### Example 1: Image Storage

```
Images per user: 100
Users: 100M
Average image size: 2 MB

Total storage: 100M × 100 × 2 MB = 20 PB
```

---

### Example 2: Video Storage

```
Videos per day: 1M
Average video size: 100 MB
Retention: 1 year

Total storage: 1M × 100 MB × 365 = 36.5 PB/year
```

---

### Example 3: Database Storage

```
Records: 1B
Average record size: 1 KB
Indexes: 30% overhead

Total storage: 1B × 1 KB × 1.3 = 1.3 TB
```

---

## Bandwidth Calculations

### Example 1: Video Streaming

```
Concurrent users: 10M
Average bitrate: 5 Mbps

Total bandwidth: 10M × 5 Mbps = 50 Tbps
```

---

### Example 2: File Upload

```
Uploads per day: 10M
Average file size: 10 MB

Daily bandwidth: 10M × 10 MB = 100 TB/day
Average bandwidth: 100 TB / 86,400 s = 1.16 GB/s
```

---

## Practice Problems

### Problem 1: Twitter-like System

**Requirements:**
- 300M users
- 200M daily active users
- 100 tweets/user/day
- 140 characters per tweet

**Calculate:**
1. Tweets per day
2. Storage per year
3. QPS

**Solution:**
```
1. Tweets/day: 200M × 100 = 20B tweets/day

2. Storage:
   Per tweet: 140 bytes + metadata (100 bytes) = 240 bytes
   20B × 240 bytes = 4.8 TB/day
   4.8 TB × 365 = 1.75 PB/year

3. QPS:
   20B / 86,400 = 231,481 QPS ≈ 230K QPS
   Peak: 230K × 3 = 690K QPS
```

---

### Problem 2: Instagram-like System

**Requirements:**
- 1B users
- 500M daily active users
- 2 posts/user/day
- Average image: 2 MB

**Calculate:**
1. Posts per day
2. Storage per day
3. Bandwidth for image serving

**Solution:**
```
1. Posts/day: 500M × 2 = 1B posts/day

2. Storage:
   1B × 2 MB = 2 PB/day

3. Bandwidth:
   Assume 10% of images viewed per day
   1B × 0.1 × 2 MB = 200 TB/day
   200 TB / 86,400 s = 2.3 GB/s
```

---

### Problem 3: Uber-like System

**Requirements:**
- 100M users
- 10M daily active users
- 2 rides/user/day
- Ride data: 1 KB

**Calculate:**
1. Rides per day
2. Storage per year
3. QPS

**Solution:**
```
1. Rides/day: 10M × 2 = 20M rides/day

2. Storage:
   20M × 1 KB = 20 GB/day
   20 GB × 365 = 7.3 TB/year

3. QPS:
   20M / 86,400 = 231 QPS
   Peak: 231 × 5 = 1,155 QPS ≈ 1.2K QPS
```

---

## Quick Reference

### Common Conversions

```
1 KB = 1,000 bytes
1 MB = 1,000 KB = 1,000,000 bytes
1 GB = 1,000 MB = 1,000,000,000 bytes
1 TB = 1,000 GB = 1,000,000,000,000 bytes
1 PB = 1,000 TB
```

### Time Conversions

```
1 second = 1,000 milliseconds
1 minute = 60 seconds
1 hour = 3,600 seconds
1 day = 86,400 seconds
1 year = 31,536,000 seconds ≈ 3.15 × 10^7
```

### Useful Approximations

```
π ≈ 3.14
e ≈ 2.72
log10(2) ≈ 0.3
log10(3) ≈ 0.48
```

---

## Summary

**Key Formulas:**
1. ✅ **QPS = Total requests / seconds in day**
2. ✅ **Storage = Records × Size × Retention**
3. ✅ **Bandwidth = QPS × Data size**
4. ✅ **Peak = Average × 3-5x**

**Practice:**
- Estimate before calculating
- Round numbers
- Use approximations
- Show your work

**Next Steps:**
- Review [Common Interview Questions](./27_COMMON_INTERVIEW_QUESTIONS.md)
- Practice [System Design Problems](./13_DESIGNING_URL_SHORTENER.md)
- Study [Interview Process](./25_SYSTEM_DESIGN_INTERVIEW_PROCESS.md)

---

**References:**
- [Latency Numbers](https://gist.github.com/jboner/2841832)
- [System Design Calculations](https://algomaster.io/learn/system-design)

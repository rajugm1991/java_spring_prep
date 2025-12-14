# Monolithic vs Microservices

## Table of Contents
1. [Introduction](#introduction)
2. [Monolithic Architecture](#monolithic-architecture)
3. [Microservices Architecture](#microservices-architecture)
4. [Comparison](#comparison)
5. [When to Use What](#when-to-use-what)
6. [Migration Strategies](#migration-strategies)
7. [Real-World Examples](#real-world-examples)

---

## Introduction

Choosing between **Monolithic** and **Microservices** architecture is one of the most important decisions in system design.

### Key Question

**Should we build one large application or many small services?**

---

## Monolithic Architecture

**Monolithic** means all components are in a single codebase and deployed together.

### Architecture

```
┌─────────────────────────────────┐
│      Monolithic Application     │
│                                 │
│  ┌──────────┐  ┌──────────┐   │
│  │   User   │  │  Order   │   │
│  │  Module  │  │  Module  │   │
│  └──────────┘  └──────────┘   │
│  ┌──────────┐  ┌──────────┐   │
│  │ Payment  │  │ Product │   │
│  │  Module  │  │  Module │   │
│  └──────────┘  └──────────┘   │
│                                 │
│  ┌──────────────────────────┐  │
│  │     Shared Database      │  │
│  └──────────────────────────┘  │
└─────────────────────────────────┘
```

### Pros

- ✅ **Simple Development:** One codebase, easy to understand
- ✅ **Easy Testing:** Run entire application locally
- ✅ **Simple Deployment:** Single deployable unit
- ✅ **ACID Transactions:** Easy across modules
- ✅ **Performance:** No network calls between modules

### Cons

- ❌ **Scaling:** Must scale entire application
- ❌ **Technology Lock-in:** Hard to use different tech stacks
- ❌ **Deployment:** Small change requires full deployment
- ❌ **Fault Isolation:** One bug can bring down entire system
- ❌ **Team Coordination:** Large teams work on same codebase

### When to Use

- Small to medium applications
- Small team
- Simple requirements
- Fast time to market needed
- Startups/MVPs

---

## Microservices Architecture

**Microservices** means application is split into independent services.

### Architecture

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│   User   │  │  Order   │  │ Payment  │
│ Service  │  │ Service  │  │ Service │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │            │            │
     └────────────┼────────────┘
                  │
          ┌───────┴───────┐
          │  API Gateway  │
          └───────┬───────┘
                  │
          ┌───────┴───────┐
          │               │
     ┌────┴────┐     ┌────┴────┐
     │  User   │     │  Order  │
     │   DB    │     │   DB    │
     └─────────┘     └─────────┘
```

### Pros

- ✅ **Independent Scaling:** Scale services independently
- ✅ **Technology Diversity:** Use best tech for each service
- ✅ **Independent Deployment:** Deploy services separately
- ✅ **Fault Isolation:** One service failure doesn't affect others
- ✅ **Team Autonomy:** Teams work independently

### Cons

- ❌ **Complexity:** Distributed system complexity
- ❌ **Network Latency:** Inter-service communication overhead
- ❌ **Data Consistency:** Hard to maintain ACID transactions
- ❌ **Testing:** Requires integration testing
- ❌ **Operational Overhead:** More services to manage

### When to Use

- Large applications
- Multiple teams
- Different scaling needs per service
- Need for technology diversity
- Mature organizations

---

## Comparison

| Aspect | Monolithic | Microservices |
|--------|-----------|---------------|
| **Development** | Simple | Complex |
| **Deployment** | Single unit | Independent |
| **Scaling** | Scale all | Scale individually |
| **Technology** | Single stack | Multiple stacks |
| **Testing** | Easy | Complex |
| **Fault Isolation** | Poor | Good |
| **Team Size** | Small | Large |
| **Startup Time** | Fast | Slower |

---

## When to Use What

### Start with Monolithic If:

1. **Small Team:** < 10 developers
2. **Simple Application:** Basic CRUD operations
3. **Uncertain Requirements:** Still figuring out features
4. **Fast MVP:** Need to ship quickly
5. **Limited Resources:** Can't manage multiple services

### Move to Microservices If:

1. **Large Team:** > 50 developers
2. **Complex Application:** Multiple domains
3. **Different Scaling Needs:** Some services need more resources
4. **Technology Requirements:** Need different tech stacks
5. **Mature Product:** Clear service boundaries

---

## Migration Strategies

### Strategy 1: Strangler Fig Pattern

Gradually replace monolithic with microservices.

```
Phase 1:
┌─────────────────┐
│   Monolith      │  ← Still handles most requests
└─────────────────┘
┌──────────┐
│ Service  │  ← New service handles specific feature
└──────────┘

Phase 2:
┌─────────────────┐
│   Monolith      │  ← Reduced functionality
└─────────────────┘
┌──────────┐  ┌──────────┐
│ Service  │  │ Service  │  ← More services
└──────────┘  └──────────┘

Phase 3:
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Service  │  │ Service  │  │ Service  │  ← All microservices
└──────────┘  └──────────┘  └──────────┘
```

### Strategy 2: Database per Service

Extract service with its own database.

```
Before:
┌─────────────────┐
│   Monolith      │
│  ┌───────────┐  │
│  │  Shared   │  │
│  │  Database │  │
│  └───────────┘  │
└─────────────────┘

After:
┌──────────┐  ┌──────────┐
│ Service  │  │ Monolith │
│  ┌────┐  │  │  ┌────┐  │
│  │ DB │  │  │  │ DB │  │
│  └────┘  │  │  └────┘  │
└──────────┘  └──────────┘
```

---

## Real-World Examples

### Example 1: Amazon's Journey

**Started:** Monolithic (1995-2001)
- Single application
- All features in one codebase

**Migrated:** Microservices (2001-2006)
- Split by business domain
- Each service owns its data
- Independent teams

**Result:**
- Faster feature development
- Independent scaling
- Better fault isolation

---

### Example 2: Netflix's Migration

**Started:** Monolithic (2007)
- Single Java application
- All features together

**Migrated:** Microservices (2009-2012)
- Split into 700+ services
- Cloud-native architecture
- Independent deployment

**Result:**
- Handles 1B+ requests/day
- 99.99% availability
- Fast feature delivery

---

### Example 3: Uber's Architecture

**Started:** Monolithic (2009)
- Single Python application
- All cities in one app

**Migrated:** Microservices (2014-2016)
- Split by domain (Rides, Eats, etc.)
- Different services for different regions
- Independent scaling

**Result:**
- Handles millions of rides/day
- Regional customization
- Better performance

---

## Summary

**Key Takeaways:**
1. ✅ **Start Monolithic:** For small teams, simple apps
2. ✅ **Migrate to Microservices:** When you outgrow monolith
3. ✅ **Use Strangler Pattern:** Gradual migration
4. ✅ **Consider Trade-offs:** Complexity vs Flexibility
5. ✅ **Learn from Others:** Amazon, Netflix, Uber

**Next Steps:**
- Learn [Event-Driven Architecture](./08_EVENT_DRIVEN_ARCHITECTURE.md)
- Understand [API Design Patterns](./09_API_DESIGN_PATTERNS.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [Microservices Guide](../ADVANCED_MICROSERVICES_GUIDE.md)
- [System Design Patterns](https://algomaster.io/learn/system-design)

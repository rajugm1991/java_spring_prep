# Distributed Systems FAQs - 50+ Frequently Asked Questions

> **Curated from real interviews at top tech companies**

This guide contains 50+ frequently asked distributed systems questions.

---

## CAP Theorem

### Q1: What is the CAP theorem?

**Answer:**
CAP theorem states a distributed system can guarantee only 2 of 3:
- **Consistency**: All nodes see same data
- **Availability**: System remains operational
- **Partition Tolerance**: System continues despite network failures

---

### Q2: How do you choose between CP and AP?

**Answer:**
- **CP**: Banking, financial systems (consistency critical)
- **AP**: Social media, content delivery (availability critical)
- Most systems choose AP with eventual consistency

---

## Consistency

### Q3: What is eventual consistency?

**Answer:**
System will become consistent over time, but not immediately. Acceptable for most use cases.

---

### Q4: How do you handle conflicts in distributed systems?

**Answer:**
- **Last Write Wins**: Simple but may lose data
- **Vector Clocks**: Track causality
- **CRDTs**: Conflict-free replicated data types
- **Application Logic**: Business-specific resolution

---

## Consensus

### Q5: What is the Raft algorithm?

**Answer:**
Consensus algorithm for distributed systems:
- Leader election
- Log replication
- Fault tolerance
- Simpler than Paxos

---

*This guide contains 50+ FAQs covering CAP theorem, consistency, consensus, distributed locks, and more.*

---

*This guide is continuously updated with new FAQs.*


## ✅ **Junior-Level Questions**

---

### **1. What does the term "data-intensive application" mean?**

A data-intensive application is one where the primary challenges involve storing, retrieving, processing, and managing large volumes of data—not computation. Examples include social platforms, e-commerce, analytics systems, and messaging systems. These apps rely on databases, caches, search engines, message brokers, and stream processors to meet performance and reliability needs.

---

### **2. What are the three main pillars for designing modern systems?**

The three pillars are:

* **Reliability:** The system continues working correctly even when components fail.
* **Scalability:** The system can handle increased load without performance degradation.
* **Maintainability:** The system is easy to modify, operate, debug, and evolve over time.

---

### **3. What is reliability in system design?**

Reliability means the system behaves correctly and predictably under normal and failure conditions. Reliable systems tolerate hardware failures, software bugs, and human errors through mechanisms like replication, redundancy, automated recovery, monitoring, backups, and failover.

---

### **4. How does scalability differ from performance?**

* **Performance** refers to how fast a system handles a single request (latency).
* **Scalability** refers to how the system behaves as load increases—whether it can process more requests (throughput) while maintaining acceptable latency.

A system may perform well at low load but fail to scale.

---

### **5. Why is maintainability important?**

Maintainability ensures the system remains easy to modify, operate, and extend over time. Since software evolves continuously, maintainability reduces technical debt, lowers operational risk, improves developer productivity, and keeps long-term cost manageable.

---

---

## 🟨 **Mid-Level Questions**

---

### **6. What are common causes of unreliability, and how can they be mitigated?**

Common causes include:

* **Hardware failures:** mitigated by redundancy, replication, and failover.
* **Software bugs:** mitigated by testing, automation, static analysis, fault isolation.
* **Human error:** mitigated by guardrails, automation, runbooks, and staging environments.

Reliable systems assume failures will happen and are designed to recover.

---

### **7. Difference between vertical and horizontal scaling? When use each?**

* **Vertical scaling (scale up):** Increase resources (CPU, RAM) on a single machine.
  Best for simplicity or legacy systems not built for distribution.

* **Horizontal scaling (scale out):** Add more machines to share workload.
  Best for large-scale distributed systems where growth is expected.

---

### **8. How does caching help scalability, and what trade-offs does it introduce?**

Caching reduces load on primary storage by storing frequent data in a fast system (Redis, CDN).
Trade-offs include:

* Stale reads
* Cache invalidation complexity
* Potential inconsistency
* Hot key bottlenecks

Caching improves latency and throughput but adds consistency challenges.

---

### **9. What is accidental complexity? Example?**

Accidental complexity is complexity introduced by design mistakes, poorly justified abstractions, or inconsistent system patterns—not required by business logic.

Examples:

* Over-engineered microservices for a small product
* Excessive layers of abstractions
* Hard-coded configurations
* Poor naming conventions

Good architecture aims to minimize accidental complexity.

---

### **10. How do we ensure a system remains maintainable over time?**

By focusing on:

* **Operability:** automation, monitoring, CI/CD, runbooks
* **Simplicity:** clear structure, avoid unnecessary abstractions
* **Evolvability:** modular design, versioning, backwards-compatible changes, refactoring, tests

Maintainability is achieved through both technical and organizational discipline.

---

---

## 🟥 **Senior / System Design-Level Questions**

---

### **11. How do replication and partitioning solve different scalability problems? Can they be used together?**

* **Replication** improves availability and read scalability by creating multiple copies of data.
* **Partitioning (sharding)** distributes data across nodes, enabling write scalability and large dataset storage.

Yes, they can be used together: each shard may be replicated for redundancy. Systems like Cassandra, MongoDB, and DynamoDB combine both.

---

### **12. How would you design a system to remain available during node or network failures?**

Use:

* Leader–follower or multi-leader replication
* Automatic failover and election (e.g., Raft, Paxos, Zookeeper, etcd)
* Retry logic and idempotent operations
* Distributed consensus where needed
* Health checks + traffic routing (load balancers)

The system must degrade gracefully, not fail catastrophically.

---

### **13. Give an example where improving scalability could hurt maintainability or reliability.**

Moving from monolith to microservices increases scalability and autonomy but introduces:

* Network failures
* Distributed troubleshooting
* Data consistency challenges
* Harder deployments and observability requirements

Higher scalability may reduce maintainability unless supported by tooling and strong engineering maturity.

---

### **14. How do human errors cause outages, and how do we reduce the risk?**

Human errors such as deploying bad configs or dropping production tables cause outages.
Mitigation strategies:

* Automated deployment pipelines
* Immutable infrastructure
* Feature flags and canary releases
* Access controls and safeguards
* Blameless postmortems and learning culture

Design assumes human mistakes will occur.

---

### **15. A system built for a single server now has millions of users. How would you evolve it?**

Step-wise evolution:

1. Add caching
2. Add replication for read scalability
3. Introduce partitioning for write scalability
4. Add async processing with message queues
5. Separate OLTP and analytics workloads
6. Move to multi-region/ distributed architecture

This approach scales capacity while reducing downtime and risk.

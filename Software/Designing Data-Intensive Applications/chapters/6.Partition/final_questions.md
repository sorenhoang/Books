## **Interview Questions for Chapter 6 — Partitioning**

These interview questions cover fundamental concepts, trade-offs, design decisions, and real-world system behavior related to **partitioning**, based on Designing Data-Intensive Applications Chapter 6.
Answers are written clearly and deeply, suitable for **senior-level** interviews.

---

# **I. Core Concepts of Partitioning**

### **1. What is partitioning and why is it needed?**

**Answer:**
Partitioning (sharding) means splitting a large dataset across multiple machines so no single node becomes a bottleneck. It's needed because:

* a single machine cannot store all data,
* CPU, RAM, or disk throughput gets saturated,
* read/write throughput must scale horizontally,
* fault isolation improves system resilience.

Partitioning provides scalability, parallelism, and better resource utilization.

---

### **2. What are the main partitioning strategies?**

**Answer:**

1. **Range Partitioning** — partitions keys into ordered ranges.

   * Pros: supports range scans; preserves locality.
   * Cons: hotspot risk if inserts are sequential.

2. **Hash Partitioning** — apply `hash(key) → partition`.

   * Pros: good load balancing; avoids hotspots.
   * Cons: breaks ordering; rebalancing expensive.

3. **Consistent Hashing** — keys and nodes placed on a hash ring.

   * Pros: minimal data movement when nodes join/leave.
   * Cons: requires vnodes; uneven distribution if poorly configured.

Each strategy is chosen based on workload patterns and system constraints.

---

# **II. Partitioning + Replication**

### **3. How do partitioning and replication interact in a distributed database?**

**Answer:**
Each partition is replicated across multiple nodes. That means:

* every partition has its own **replication group**,
* each group has a **leader** (or quorum replicas in leaderless systems),
* replication, failover, and logs operate *per partition*, not cluster-wide.

This makes the system scalable and fault-tolerant: only the affected partitions require failover during node failure.

---

### **4. Why do systems prefer many small partitions instead of a few large ones?**

**Answer:**
Small partitions provide:

* finer-grained load balancing,
* more flexible rebalancing,
* faster recovery on failure,
* parallelism across many threads and nodes.

Large partitions make rebalancing slow and increase recovery times.

---

# **III. Partitioning and Secondary Indexes**

### **5. What is the difference between local and global secondary indexes?**

**Answer:**

| Type             | How It Works                                      | Pros                                 | Cons                                                 |
| ---------------- | ------------------------------------------------- | ------------------------------------ | ---------------------------------------------------- |
| **Local Index**  | index stored inside each partition                | fast writes; simple                  | queries require scatter/gather; no global uniqueness |
| **Global Index** | separate partitioned index spanning whole dataset | efficient lookups; global uniqueness | complex writes; may need 2PC or eventual consistency |

Global indexes improve read performance but significantly complicate write paths.

---

### **6. Why are global secondary indexes expensive to maintain?**

**Answer:**
Because a write must update:

1. **the primary partition** (hash primary key), and
2. **the index partition** (hash secondary key).

This cross-partition write requires:

* distributed transactions (2PC), or
* asynchronous propagation with eventual consistency.

Both options increase latency, write amplification, and failure modes.

---

# **IV. Rebalancing**

### **7. What triggers partition rebalancing?**

**Answer:**
Rebalancing happens when:

* new nodes added (scale out),
* nodes removed (failures or scale in),
* partitions too large or too hot,
* replication factor changes,
* skew appears in key distribution or access patterns.

Rebalancing keeps the cluster healthy and evenly loaded.

---

### **8. How does consistent hashing make rebalancing easier?**

**Answer:**
In consistent hashing:

* adding a node only affects keys in adjacent ranges,
* removing a node only redistributes its portion of keys.

Movement is limited to **~1/N** of the keys, avoiding cluster-wide reshuffling.
This drastically reduces data movement and rebalancing time.

---

### **9. What is range splitting and why is it useful?**

**Answer:**
Range splitting divides an oversized or hot range into smaller subranges:

```
[A–Z] → [A–M] + [M–Z]
```

It’s useful because:

* prevents hotspots,
* keeps partitions at manageable sizes,
* enables fine-grained movement to balance nodes.

Used in systems like Bigtable, HBase, and CockroachDB.

---

### **10. What makes rebalancing a risky operation?**

**Answer:**
Rebalancing can:

* overwhelm network/disk bandwidth,
* cause latency spikes,
* interfere with ongoing reads/writes,
* lead to temporary routing inconsistencies,
* risk lost writes if cutover isn’t coordinated correctly.

Systems throttle rebalancing and use snapshot + log replay + fencing tokens to ensure correctness.

---

# **V. Request Routing**

### **11. What are the main routing strategies?**

**Answer:**

1. **Client-side routing**
   Clients compute partition and talk directly to nodes.

2. **Routing tier (e.g., mongos)**
   A router node forwards queries to correct partition.

3. **Decentralized routing**
   Any node can forward requests to the appropriate partition.

Each offers trade-offs in latency, simplicity, and reliability.

---

### **12. What is a partition map, and how is it maintained?**

**Answer:**
Partition map = metadata describing:

* partition boundaries,
* replica locations,
* leaders,
* health status.

Maintained via:

* **gossip** (Cassandra, Dynamo),
* **config servers** (MongoDB),
* **consensus protocols** (CockroachDB, Spanner).

Routing correctness depends on map accuracy.

---

### **13. How does routing handle node failures?**

**Answer:**

* detect failure (heartbeats, gossip, failure detectors),
* reroute requests to another replica,
* elect new leader if necessary,
* update routing metadata,
* retry or repair incomplete writes.

Routing must seamlessly adapt to failures to maintain high availability.

---

### **14. How does routing work during rebalancing?**

**Answer:**
Routing must:

* ensure writes go to the new partition location,
* optionally dual-write during cutover,
* prevent old leaders from accepting writes (fencing),
* handle mixed metadata versions until all clients update.

Routing during rebalancing is one of the hardest correctness challenges in distributed systems.

---

# **VI. Hotspots and Load Balancing**

### **15. What causes hotspots in partitioned databases?**

**Answer:**
Hotspots appear when:

* keys are sequential (timestamps, auto-increment IDs),
* one key/user is disproportionately popular,
* bad partitioning logic (e.g., bad hash function),
* uneven traffic patterns,
* insufficient partition splitting.

Hot partitions become bottlenecks for the entire cluster.

---

### **16. How can hotspots be mitigated?**

**Answer:**

* **Hash partitioning** (randomizes distribution)
* **Key salting** (spread writes across fixed buckets)
* **Adaptive range splitting**
* **Vertical partitioning** (separate hot fields)
* **Caching** at front layers
* **Write absorbing structures** (LSM-tree buffers)

Systems must detect hotspots early and react automatically.

---

# **VII. Designing for Scale**

### **17. Why is it important to decouple routing metadata from data storage?**

**Answer:**
Because:

* metadata changes frequently (due to failures, rebalancing)
* data volumes are huge

Separating metadata allows routing to be fast, lightweight, and strongly/weakly consistent independent of data.

This design pattern underlies Elasticsearch, Kafka, CockroachDB, and many others.

---

### **18. What is the role of virtual nodes (vnodes)?**

**Answer:**
Vnodes improve:

* load balancing,
* rebalancing flexibility,
* fault tolerance.

Instead of one large partition per node, each node stores many small vnodes. When a node joins/leaves, only a subset of vnodes move, minimizing data movement.

---

### **19. Compare request routing in Cassandra vs MongoDB.**

**Answer:**

**Cassandra**

* Client-side routing.
* Gossip distributes metadata.
* Any node can act as coordinator.
* Leaderless reads/writes using quorums.

**MongoDB**

* Routing tier (`mongos`) performs query routing.
* Config servers maintain metadata.
* Writes go to shard leaders.
* Secondary indexes available on shards.

Cassandra favors decentralization and high availability; MongoDB favors centralized metadata and easier operational model.

---

### **20. What happens when the number of partitions changes in modulo-based hashing?**

**Answer:**
Nearly **all keys must be moved** because:

```
partition = hash(key) % N
```

When N changes → partition assignments shift for most keys.
This causes:

* huge data movement,
* rebalancing storms,
* long maintenance windows.

This is why modern systems avoid modulo hashing.

---

# **VIII. Scenario-Based / System Design Questions**

### **21. Design a distributed database that supports range scans and avoids hot partitions.**

**Answer:**

* Use **range partitioning** for efficient scans.
* Implement **automatic splitting** of hot ranges.
* Ensure **metadata servers** maintain partition boundaries.
* Use **adaptive routing** to nearest replica.
* Monitor skew and redistribute as needed.

This approach matches systems like Bigtable, CockroachDB, and HBase.

---

### **22. Your cluster has one partition that is extremely hot. How do you fix it?**

**Answer:**
Solutions:

* Split the hot range into smaller partitions.
* Redistribute partitions across less-loaded nodes.
* Apply salting (if applicable).
* Use caching in front to absorb hot reads.
* Offload writes using log-structured storage.
* Redirect hot user's data into isolated partition.

Naturally depends on storage engine and index/model design.

---

### **23. How do you guarantee consistency when migrating a partition to a new node?**

**Answer:**

Steps:

1. Freeze a consistent snapshot.
2. Copy snapshot to new node.
3. Stream WAL/log entries to catch up.
4. Apply fencing token to old leader.
5. Atomically update routing metadata.
6. Switch leader to new location.
7. Retire old partition safely.

This approach ensures no writes are lost or misrouted.

---

### **24. When would you choose global secondary indexes over local ones?**

**Answer:**
When you need:

* fast point lookups across the entire dataset,
* global uniqueness constraints,
* efficient filtering on non-primary attributes.

Examples:

* querying by email, username, product category.

Use cases with high write throughput may not be suitable due to overhead.

---

### **25. What consistency challenges arise with scatter/gather queries?**

**Answer:**

* Different partitions may be at different replication states → inconsistent results.
* Some partitions may be temporarily unavailable → partial responses.
* Merge phase must reconcile conflicting versions.

Most systems choose eventual consistency for global queries.

---

# **IX. Final Takeaways**

These questions cover all crucial concepts:

* Why partitioning is necessary
* How partitioning strategies differ
* How secondary indexes complicate writes
* How rebalancing works safely
* How request routing works in real systems
* How to handle hotspots and skew
* How metadata consistency ties everything together

Partitioning is the backbone of modern distributed databases—and one of the most technically challenging areas of system design.


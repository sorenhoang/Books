## **Interview Questions for Chapter 5 — Replication**
---

# **I. Fundamentals of Replication**

### **1. What is database replication and why do we need it?**

**Answer:**
Replication is the process of **maintaining multiple copies of the same data** across different nodes in a distributed system. Its core purposes are:

* **High Availability**: If one node fails, others can take over.
* **Fault Tolerance**: Hardware failures, crashes, network partitions don’t wipe out data.
* **Read Scalability**: Replicas can serve read queries to distribute load.
* **Latency Reduction**: Clients read/write to closer replicas.
* **Durability**: Multiple copies reduce the risk of data loss.

Replication is the foundation of distributed systems because **hardware fails all the time**, and data must remain available.

---

### **2. What are the main replication models?**

**Answer:**

#### ✔ **Leader–Follower**

* Single leader handles writes.
* Followers replicate the log.
* Simple, consistent, widely used.

#### ✔ **Multi-Leader**

* Multiple leaders accept writes.
* Writes are replicated between leaders.
* Good for geo-distributed writes, but **conflicts possible**.

#### ✔ **Leaderless (Dynamo-style)**

* Any replica can accept reads/writes.
* **Quorum** determines correctness.
* Very high availability and throughput, but **eventual consistency**.

Each model embodies different trade-offs between **consistency, availability, latency, conflict handling**.

---

### **3. What is the replication log? Why is it important?**

**Answer:**
The replication log is an **ordered, append-only sequence of write operations** that records how the database changes over time. It ensures:

* **Same execution order** on all replicas.
* **Crash recovery** (replay log).
* **Replication consistency** (followers apply logs).
* **CDC integration** (external pipelines).

Without a log, replicas could apply writes in **different orders**, leading to **divergent state**.

---

# **II. Leader–Follower Replication**

### **4. Explain leader–follower replication in detail.**

**Answer:**

* Client sends write to **leader**.
* Leader records write into its **log**, applies it locally.
* Followers **pull or receive** log entries.
* Followers **apply** writes in same order.

Followers can serve reads. If async, follower reads may be stale; if sync, writes wait for follower acknowledgment.

This model provides **a single source of truth**, reducing conflict complexity.

---

### **5. What is synchronous vs asynchronous replication?**

**Answer:**

#### **Synchronous**

* Leader waits for **follower acknowledgment** before responding.
* Strong durability and no lost writes on failover.
* Higher latency, lower availability.

#### **Asynchronous**

* Leader acknowledges **immediately after local write**.
* Faster writes, better availability.
* Risk of **lost writes** if leader fails before replication.

Trade-off:

> **Sync = safety over availability, Async = availability over safety**

---

### **6. Why can writes be lost during failover?**

**Answer:**
In **asynchronous replication**, the leader acknowledges a write **before** followers have received it. If the leader crashes immediately thereafter:

* The new leader might **not have the write**.
* Client sees success, but data disappears → **failover-induced data loss**.

This is why financial systems often use **synchronous or quorum replication**.

---

### **7. How are new followers deployed?**

**Answer:**

1. Leader takes a **consistent snapshot**.
2. Snapshot is transferred to the follower.
3. Follower **replays the log** from snapshot’s position.
4. When caught up, it switches to **live streaming**.

Snapshot alone is **not enough**, because writes continue during snapshot creation.

---

### **8. What happens when a follower goes down?**

**Answer:**
Follower failures are easy:

* Leader continues taking writes.
* Follower catches up via **log replay** on restart.
* If logs are truncated → a new snapshot is needed.

No failover required because leader remains alive.

---

### **9. How does leader failover work?**

**Answer:**
Leader failures are more complex:

* Cluster detects failure (timeouts/heartbeats).
* Followers elect a **new leader**.
* Redirect writes to new leader.
* Former leader must **discard uncommitted state** if it returns.

Consensus protocols like **Raft** ensure **no split-brain** and that **only one leader** is active.

---

# **III. Multi-Leader Replication**

### **10. What is multi-leader replication and when is it useful?**

**Answer:**
Multiple nodes accept writes and replicate changes to each other. It’s useful for:

* **Multi-region low-latency writes**
* **Offline / mobile sync**
* **Collaborative apps**
* **Branch office deployments**

Because each leader accepts writes independently, **conflicts can occur** if two leaders update the same record.

---

### **11. How do you resolve conflicts in multi-leader replication?**

**Answer:**
Options include:

* **Last Write Wins (LWW)** by timestamp
* **Custom business logic** (merge fields)
* **Causal merge** (sum counters)
* **Vector clocks** to detect concurrency
* **CRDTs** (conflict-free replicated data types)

There is no universal solution—**domain knowledge** defines correct behavior.

---

### **12. Why do multi-leader systems often use eventual consistency?**

**Answer:**
Because writes are **accepted independently** by different leaders and **replicated asynchronously**, different leaders may momentarily have different values. They converge **eventually** after conflict resolution or merging.

---

# **IV. Leaderless Replication & Quorums**

### **13. How does leaderless replication work?**

**Answer:**
Any node can accept reads/writes. The system uses **quorum voting**:

* write is committed when **W** replicas respond,
* read is valid when **R** replicas respond.

Condition for strong consistency:

> **R + W > N**

Guarantees at least **one replica** has both read & write.

Example:

* N = 3
* W = 2
* R = 2 → strong consistency

---

### **14. What are R, W, and N?**

**Answer:**

* **N** → total replicas storing a key
* **W** → # of replicas required for write success
* **R** → # required to return value

Quorum logic:

* large W → durable writes
* large R → fresher reads
* small W or R → faster but stale

---

### **15. Explain read repair and hinted handoff.**

**Answer:**

#### **Read Repair**

During a read:

* coordinator sees different versions,
* picks correct value,
* writes back to **stale replicas** in background.

Ensures convergence.

#### **Hinted Handoff**

If a replica is down:

* neighbor stores write **on its behalf**,
* delivers write when the replica recovers.

Avoids write loss during partial failures.

---

### **16. What consistency guarantees do quorum systems provide?**

**Answer:**
Depends on R and W:

* **R + W > N** → strong consistency
* **R + W ≤ N** → eventual consistency
* **W = majority** → safe commit
* **R = majority** → read correctness

Quorums allow **tunable consistency**: clients select consistency level **per request**.

---

# **V. Trade-offs in Replication**

### **17. Why does the CAP theorem matter for replication?**

**Answer:**
During a network partition, a system must choose **Consistency or Availability**, not both.

* **CP systems** → reject writes to preserve consistency
* **AP systems** → continue accepting writes but data diverge, later converge

Replication models inherently **choose one side** during failures.

**PACELC** extends CAP:

> If Partition → choose A or C
> Else → choose Latency or Consistency

Meaning trade-offs exist even without failure.

---

### **18. What are the trade-offs of synchronous vs asynchronous replication?**

**Answer:**

| Mode      | Latency | Availability | Durability | Risk        |
| --------- | ------- | ------------ | ---------- | ----------- |
| Sync      | High    | Lower        | Strong     | No loss     |
| Async     | Low     | High         | Weak       | Lost writes |
| Semi-sync | Medium  | Medium       | Medium     | Rare loss   |

Synchronous favors **data integrity**, asynchronous favors **performance and uptime**.

---

### **19. Why not always use strong consistency?**

**Answer:**
Strong consistency requires:

* global coordination,
* synchronous replication,
* blocking writes during failures or partitions.

This increases:

* latency,
* failure sensitivity,
* operational complexity.

Many applications **do not need strict consistency** and benefit from **lower latency and higher availability**.

Examples:

* likes, counters, feeds,
* analytics events,
* caching layers.

---

### **20. How do modern systems combine replication strategies?**

**Answer:**
Modern databases use **hybrid models**:

* **consensus per shard** (Raft),
* **followers for reads**,
* **logical logs for CDC**,
* **geo-replication for latency**.

Examples:

* CockroachDB,
* Spanner,
* TiDB.

They look like **multi-leader globally**, but ensure **one leader per key** to avoid conflicts.

Thus:

> **global write scalability + strong consistency**
> without application-level conflict resolution.

---

# **VI. Scenario / System Design Questions**

Now the style changes to **open-ended design scenarios**, very common in senior interviews.

---

### **21. If you run a global banking system, which replication model do you choose? Why?**

**Answer:**

* Choose **leader-based with consensus** (Raft/Paxos) per partition.
* Requires **strong consistency**, **strict ordering**, **no lost writes**, **no conflicts**.
* Banking is **CP** domain: reject writes during partitions rather than risk inconsistency.

Geo-replicas can serve **read-only** workloads.

---

### **22. How would you design a multi-region social feed system?**

**Answer:**

* Choose **leaderless or multi-leader**, because:

  * writes from users are **independent**,
  * staleness is **acceptable**,
  * latency matters more than strict ordering,
  * mergeable operations (likes, follows) fit CRDT/counter models.

Use:

* **R=1, W=1** for low latency,
* **anti-entropy repair** ensures convergence.

---

### **23. How do you enforce unique constraints in multi-leader replication?**

**Answer:**

Hard problem — require **coordination**:

Options:

* **shard by unique key**, one leader per key,
* **consensus check** before commit,
* **global index** with atomic updates,
* **materialized unique constraints** via dedicated service.

Many systems simply **avoid** global uniqueness in multi-leader setups.

---

### **24. What happens if two leaders update the same record simultaneously?**

**Answer:**
A **write conflict** occurs. Resolution strategies:

* Timestamps (`Last Write Wins`)
* Merge logic (`sum`, `union`, patch)
* Expose conflict to the client
* Use **vector clocks** to detect concurrency
* Use **CRDTs** to auto-merge

Without correct logic, data becomes **inconsistent**.

---

### **25. How does Raft differ from quorum leaderless systems?**

**Answer:**

* **Raft (consensus):**

  * One leader per log,
  * Majority quorum **for every write**,
  * **Global ordering** of writes,
  * Rejects operations without quorum,
  * **Strong consistency** (CP).

* **Leaderless (Dynamo):**

  * **No permanent leader**,
  * Writes accepted with **partial quorums**,
  * Allows conflicting writes,
  * **Eventual consistency** (AP),
  * Uses **vector clocks**, **repair mechanisms**.

Consensus avoids conflicts by **preventing concurrent writes**.

---

### **26. When would you use eventual consistency intentionally?**

**Answer:**
Whenever **low latency and high availability** are more important than strict ordering:

* social feed timelines,
* notification badges,
* user preference sync,
* logs/metrics ingestion,
* IoT data streams,
* shopping carts (mergeable sets).

The cost of stale data is low, while performance benefits are large.

---

# **VII. Final Summary for Interview**

Key points to remember:

1. **Replication = core to distributed systems**

   * availability, durability, scalability.

2. **Leader–Follower is simple**

   * single writer, no conflicts.

3. **Asynchronous vs synchronous**

   * choose between durability and latency.

4. **Multi-leader enables geo-writes**

   * requires conflict resolution.

5. **Leaderless uses quorums**

   * any replica can accept writes,
   * eventual consistency unless R+W>N.

6. **CAP & PACELC apply**

   * consistency vs availability,
   * latency vs consistency.

7. **No perfect solution**

   * models serve different domain needs.

Senior engineers must **evaluate requirements**, understand **failure modes**, and **pick the right model** rather than trying to force one model everywhere.
## **Interview Questions for Chapter 7 — Transactions**

These questions cover all major transactional concepts from Chapter 7: ACID, weak isolation, anomalies, serializable isolation, MVCC, SSI, distributed transactions, and performance trade-offs.
Answers are written clearly and deeply — suitable for **Senior Backend / Distributed Systems interviews**.

---

# 🔷 **Section 1 — ACID & Transaction Basics**

### **1. What is a transaction and why do databases use them?**

A **transaction** is a group of read/write operations treated as a single atomic unit. Databases use transactions to guarantee correctness even when:

* concurrency occurs,
* servers crash,
* partial failures happen,
* writes need to be consistent and durable.

Transactions protect against corruption and simplify application logic.

---

### **2. What do the ACID properties mean?**

* **Atomicity:** all-or-nothing execution — no partial results.
* **Consistency:** application-level invariants always hold before/after the transaction.
* **Isolation:** concurrent transactions do not interfere in ways that violate correctness.
* **Durability:** committed data survives crashes via WAL, replication, etc.

---

### **3. How does atomicity differ from isolation?**

* **Atomicity:** protects against *failures inside a single transaction*.
* **Isolation:** protects against *interference from other concurrent transactions*.
  They solve completely different problems.

---

### **4. What does durability rely on internally?**

* Write-ahead logs (WAL)
* Replica acknowledgments
* Checkpoints
* Persistent storage guarantees

Once committed, the database must be able to recover all changes.

---

# 🔷 **Section 2 — Concurrency Anomalies**

### **5. Name and explain common concurrency anomalies.**

* **Dirty read:** reading uncommitted data.
* **Dirty write:** overwriting another uncommitted write.
* **Read skew:** reading two related values at different logical times.
* **Lost update:** two transactions overwrite each other's changes.
* **Write skew:** two transactions read overlapping data but update disjoint rows → invariant violation.
* **Phantom reads:** query results change because rows were inserted/deleted by another transaction.

---

### **6. Which anomaly is the most dangerous under Snapshot Isolation?**

**Write skew.**
Snapshot Isolation allows two transactions to make valid changes independently that *violate a higher-level invariant*.

Example: two doctors turning off their "on-call" flag, leaving nobody on call.

---

# 🔷 **Section 3 — Weak Isolation Levels**

### **7. What anomalies does Read Uncommitted allow?**

* Dirty reads
* Dirty writes
* Read skew
* Lost updates
* Write skew
* Phantoms
  Effectively near-zero protection.

---

### **8. What guarantees does Read Committed provide?**

Prevents:

* dirty reads
* dirty writes

Still allows:

* read skew
* write skew
* lost updates
* phantom reads

Most common default level (PostgreSQL, Oracle, SQL Server).

---

### **9. How does Snapshot Isolation (SI) work?**

* Transaction reads from a **consistent snapshot** of the database
* Uses **MVCC** (multi-version concurrency control)
* Writers create new versions instead of blocking readers
* Prevents read skew and dirty reads
* Still allows write skew

---

### **10. Why does Snapshot Isolation still allow write skew?**

SI does *not* detect conflicts based on read sets — only write/write conflicts.
Two concurrent transactions reading the same data but writing different rows won’t conflict.

This can break invariants even though MVCC ensures consistent snapshots.

---

### **11. What are phantom reads and why are they hard to prevent?**

A transaction re-runs a query and sees a *different set of rows* because another transaction inserted or deleted matching rows.

Hard to prevent because:

* it requires locking **ranges** of keys, not individual keys
* range locking is expensive
* often requires predicate locks or indexing tricks

---

### **12. How do databases prevent lost updates?**

* SELECT … FOR UPDATE (pessimistic locking)
* Optimistic version checking
* Atomic compare-and-set operations
* Serializable isolation level

---

# 🔷 **Section 4 — Serializable Isolation**

### **13. What does serializable isolation guarantee?**

The strongest isolation level:

> The execution of transactions must be equivalent to *some* sequential ordering of those transactions.

All anomalies (write skew, phantoms, lost updates, etc.) are prevented.

---

### **14. What are the main ways databases implement serializable isolation?**

1. **Strict Two-Phase Locking (2PL):**

   * Uses shared and exclusive locks
   * Prevents anomalies but causes blocking and deadlocks

2. **Serializable Snapshot Isolation (SSI):**

   * Used by PostgreSQL
   * Optimistic, MVCC-based
   * Detects dangerous dependency cycles
   * Aborts one transaction when necessary

3. **Deterministic Execution (Calvin-style):**

   * Pre-determines transaction order
   * Executes deterministically
   * Avoids concurrency conflicts entirely

---

### **15. What is a “dangerous structure” in SSI?**

SSI tracks dependency edges between transactions.
A cycle such as:

```
T1 →rw T2
T2 →rw T3
T3 →rw T1
```

is unsafe.
If a cycle is detected → **abort one transaction** to restore serializability.

---

### **16. How does Serializable Snapshot Isolation differ from plain Snapshot Isolation?**

* SI gives consistent snapshots but *does not detect dangerous write skew*.
* SSI builds on SI by tracking dependencies and aborting transactions that could cause anomalies.
* SSI = SI + conflict detection.

---

### **17. Why is serializable isolation expensive?**

* More metadata tracking
* More locking (in pessimistic systems)
* More aborts (in optimistic systems)
* More coordination, especially across shards
* Preventing phantoms requires predicate or range locks

Serializable adds CPU, memory, and latency overhead.

---

# 🔷 **Section 5 — MVCC & Performance**

### **18. What is MVCC and why is it useful?**

Multi-Version Concurrency Control:

* Keeps multiple versions of a row
* Readers see consistent snapshots
* Writers don’t block readers and vice versa
* Enables Snapshot Isolation and SSI

MVCC improves read-heavy workloads significantly.

---

### **19. What is the downside of MVCC?**

* Need to store old versions
* Need garbage collection (vacuuming in PostgreSQL)
* Potential bloat
* More memory and storage overhead

---

### **20. How do distributed systems implement serializable isolation?**

They require:

* global ordering of transactions (timestamps or deterministic sequencing)
* multi-shard conflict detection
* distributed commit protocols (2PC)
* consensus (Raft/Paxos) for replication
* fencing tokens for leader changes

Examples:

* Google Spanner uses TrueTime timestamps + commit wait
* CockroachDB uses hybrid logical clocks + MVCC

---

# 🔷 **Section 6 — Distributed Transactions**

### **21. What is the role of Two-Phase Commit (2PC) in distributed transactions?**

2PC ensures atomicity across multiple participants.
Phases:

1. **Prepare**: ask all nodes if they can commit
2. **Commit**: commit if everyone agreed

Downside:

* blocking protocol
* coordinator failures cause transaction freezing
* adds latency

---

### **22. Why is achieving serializable isolation across shards harder than on a single machine?**

Because it requires:

* cross-shard communication
* detecting conflicts across distributed partitions
* handling network delays and partitions
* global timestamps or logical ordering
* preventing inconsistencies from replication lag

Serializable distributed transactions often impose significant overhead.

---

# 🔷 **Section 7 — Application-Level Questions**

### **23. When should an application use transactions?**

* multi-step operations
* maintaining invariants (balances, quotas, inventory)
* when correctness matters more than max throughput
* when operations must be atomic, isolated, and durable

Not needed when:

* operations are idempotent
* eventual consistency is fine
* data can be reconstructed or is ephemeral

---

### **24. Give an example of an invariant that write skew can violate.**

“In a hospital, at least one doctor must be on call.”

Two transactions turn off different doctors → invariant violated despite no direct conflict.

---

### **25. How can you avoid write skew without switching to serializable isolation?**

* Explicit locking of the rows involved
* Using SELECT FOR UPDATE on the read set
* Application-layer mutex
* Rewriting logic to avoid multi-row invariants

---

### **26. What workload characteristics make serializable isolation a poor choice?**

* high write contention
* long-running transactions
* hot keys
* multi-partition updates
* low-latency requirements where any blocking is harmful

---

# 🔷 **Section 8 — Summary Questions**

### **27. What are the three key trade-offs when choosing an isolation level?**

1. **Correctness**
2. **Performance**
3. **Complexity for developers**

---

### **28. If your system must ensure invariants across multiple rows, which isolation level is required?**

**Serializable isolation** (strictest)
OR
careful use of locks to emulate it.

---

### **29. Why is optimistic concurrency better for read-heavy workloads?**

Reads don't block writes; writes don't block reads.
Conflicts (and aborts) are rare → high throughput.

---

### **30. What’s the difference between serializable isolation and linearizability?**

* **Serializable**: applies to multi-operation *transactions*.
* **Linearizability**: applies to *single operations* and respects real-time order.

Serializable ≠ real-time consistent unless explicitly designed (e.g., Spanner).
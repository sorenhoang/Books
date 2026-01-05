## **Interview Questions for Chapter 8 — The Trouble with Distributed Systems (WITH ANSWERS)**
---

# 🔷 **Section 1 — Partial Failures**

### **1. What is a partial failure, and why does it make distributed systems hard?**

A partial failure occurs when **some components fail while others continue operating**. Unlike a single-machine system, failure is not all-or-nothing. This makes systems hard because:

* failure detection is ambiguous,
* nodes see different system states,
* incorrect assumptions are unavoidable,
* recovery decisions must be made under uncertainty.

Partial failures are the defining challenge of distributed systems.

---

### **2. Why doesn’t the concept of “up or down” work in distributed systems?**

Because:

* a node may be slow, not dead,
* the network may be partitioned,
* messages may be delayed or dropped.

From another node’s perspective, **silence is ambiguous**. There is no reliable way to tell whether a node has crashed or is temporarily unreachable.

---

### **3. Why is failure detection impossible to do perfectly?**

Because in an asynchronous system:

* message delays are unbounded,
* clocks are unreliable,
* nodes fail independently.

A timeout is only a guess, not proof of failure. This is formalized by the **FLP impossibility result**.

---

# 🔷 **Section 2 — Unreliable Networks**

### **4. What kinds of network failures must distributed systems assume?**

Distributed systems must assume:

* packet loss,
* packet delay,
* packet duplication,
* packet reordering,
* network partitions.

Any of these can happen at any time.

---

### **5. Why does TCP not solve distributed system reliability problems?**

TCP ensures:

* reliable byte streams,
* retransmission,
* in-order delivery.

But it does **not** guarantee:

* bounded latency,
* timely failure detection,
* exactly-once semantics.

Applications must still implement timeouts, retries, and idempotency.

---

### **6. Why are retries dangerous?**

Retries can cause:

* duplicate operations,
* retry storms,
* cascading failures,
* overload amplification.

Without idempotency or deduplication, retries can corrupt data or collapse systems.

---

### **7. Why is exactly-once delivery impractical?**

Because:

* sender may not know whether receiver processed a message,
* retries can duplicate messages,
* failures can happen at any point.

Most systems implement **at-least-once delivery** and handle duplicates explicitly.

---

# 🔷 **Section 3 — Network Partitions & CAP**

### **8. What happens during a network partition?**

Nodes are alive but cannot communicate. Each side may believe the other has failed, leading to:

* conflicting leaders,
* divergent writes,
* split-brain.

Partitions force a choice between consistency and availability.

---

### **9. Explain the CAP theorem in simple terms.**

In the presence of a network partition:

* you must choose between **Consistency** and **Availability**.
* Partition tolerance is mandatory in distributed systems.

CAP is not a design choice—it’s a physical constraint.

---

### **10. How do quorums help during partitions?**

Quorums require a majority to make decisions.

Because majorities overlap:

* conflicting decisions cannot both reach quorum,
* minority partitions become unavailable,
* safety is preserved.

---

# 🔷 **Section 4 — Unreliable Clocks**

### **11. Why can’t we rely on wall-clock time in distributed systems?**

Because clocks:

* drift,
* disagree across machines,
* jump backward or forward,
* depend on unreliable synchronization.

Timestamps do not reliably reflect real event ordering.

---

### **12. What is the difference between wall-clock time and monotonic time?**

* **Wall-clock time**: human time; can move backward.
* **Monotonic time**: always moves forward; only good for measuring durations.

Monotonic clocks cannot be compared across machines.

---

### **13. What bugs can unreliable clocks cause?**

* incorrect event ordering,
* broken TTLs and leases,
* stale data overwriting new data,
* inconsistent snapshots,
* incorrect leader election.

Time-based assumptions are a common source of subtle bugs.

---

### **14. How do logical clocks help?**

Logical clocks (Lamport, vector clocks):

* capture causality without relying on physical time,
* allow safe ordering of events,
* detect concurrency.

They avoid clock skew issues.

---

### **15. What is TrueTime and why is it special?**

TrueTime (Spanner):

* returns a time interval with bounded uncertainty,
* allows waiting out uncertainty,
* enables globally consistent transaction ordering.

It requires specialized infrastructure and adds latency.

---

# 🔷 **Section 5 — Knowledge, Truth, and Lies**

### **16. Why is “truth” relative in distributed systems?**

Because:

* information is delayed,
* nodes fail independently,
* different nodes observe different events.

There is no instant, global view of the system.

---

### **17. What is split-brain and why is it dangerous?**

Split-brain occurs when:

* multiple nodes believe they are the leader,
* both accept writes.

This causes data divergence and corruption.

---

### **18. How do consensus protocols create truth?**

Consensus protocols:

* allow nodes to agree on a single value,
* ensure safety despite failures,
* manufacture shared truth from unreliable communication.

They do not eliminate uncertainty but constrain it.

---

### **19. Why do distributed systems use epochs or terms?**

Epochs/terms:

* impose ordering on leadership or decisions,
* invalidate stale leaders,
* prevent zombie processes from acting.

Higher epochs always override lower ones.

---

### **20. Why must distributed systems tolerate false beliefs?**

Because:

* nodes will inevitably have outdated or incorrect information,
* perfect synchronization is impossible.

Systems must detect mistakes and recover safely.

---

# 🔷 **Section 6 — Design & Engineering Principles**

### **21. Why must all remote calls have timeouts?**

Because:

* waiting forever blocks resources,
* network failures are indistinguishable from slowness.

Timeouts ensure progress, even if decisions may be wrong.

---

### **22. Why is idempotency so important?**

Because retries are unavoidable.

Idempotency ensures:

* repeated requests don’t cause duplicate effects,
* recovery is safe after uncertainty.

---

### **23. Why do distributed systems prefer majority-based decisions?**

Majorities:

* overlap,
* prevent conflicting decisions,
* tolerate minority failures.

They sacrifice availability on the minority side to preserve safety.

---

### **24. Why is strong coordination expensive?**

Because it requires:

* communication with many nodes,
* waiting for responses,
* handling failures and retries.

Consensus and locks reduce throughput and increase latency.

---

### **25. Why do many systems accept eventual consistency?**

Because:

* strong consistency hurts availability and latency,
* many workloads tolerate temporary inconsistency,
* systems must keep operating under failures.

Eventual consistency is a pragmatic trade-off.

---

# 🔷 **Section 7 — Scenario Questions**

### **26. A node stops responding. What assumptions should you make?**

None.

It may be:

* crashed,
* slow,
* partitioned,
* overloaded.

Design must handle all cases safely.

---

### **27. How would you prevent split-brain in a distributed database?**

* Use quorum-based leader election
* Use consensus (Raft/Paxos)
* Require majority for writes
* Use fencing tokens

---

### **28. How should systems behave when wrong assumptions are made?**

They should:

* detect inconsistencies,
* roll back or compensate,
* retry safely,
* converge to a correct state.

Recovery is more important than initial correctness.

---

# 🔷 **Section 8 — Meta-Level Understanding**

### **29. Why is Chapter 8 so important for system design interviews?**

Because it explains *why* distributed systems behave the way they do:

* why retries exist,
* why timeouts are needed,
* why consensus is expensive,
* why consistency is hard.

It shows deep understanding beyond APIs or tools.

---

### **30. Summarize Chapter 8 in one sentence.**

> **Distributed systems operate under permanent uncertainty—partial failures, unreliable communication, and incomplete knowledge—forcing designers to trade certainty for progress and correctness for availability.**

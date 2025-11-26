
## 🟩 **Junior-Level Questions (With Answers)**

---

### **1. What is the purpose of an index in a database?**

Indexes speed up data retrieval by allowing the system to locate records without scanning the entire dataset. They act like lookup tables that improve query performance at the cost of additional storage and slower writes.

---

### **2. What is the difference between row-oriented and column-oriented storage?**

* **Row-oriented storage** stores entire rows together and is optimized for OLTP (transactional workloads).
* **Column-oriented storage** stores data by columns, enabling fast scanning, compression, and analytical queries in OLAP workloads.

---

### **3. What is a Write-Ahead Log (WAL)?**

A WAL is a log where changes are recorded before being applied to the main data structure. It ensures durability so that the system can recover to a consistent state after a crash.

---

### **4. What types of queries are hash indexes best suited for?**

Hash indexes are ideal for **equality lookups** (e.g., `WHERE id = 5`) since they provide constant-time average lookup. They do **not** support efficient range queries.

---

### **5. Why are SSTables immutable?**

Immutability simplifies concurrency, allows sequential disk writes, improves compression, and enables efficient merging through compaction. It also avoids random disk writes.

---

---

## 🟨 **Mid-Level Questions (With Answers)**

---

### **6. What problem do LSM-trees solve compared to B-trees?**

LSM-trees optimize write-heavy workloads by using sequential append-only writes and batching updates in memory before flushing to disk as sorted tables. This avoids costly random writes required by B-trees.

---

### **7. Why do LSM-based systems require compaction?**

Over time, multiple SSTables accumulate outdated values and tombstones. Compaction merges tables, removes old entries, reduces read amplification, and maintains sorted structure — keeping lookups efficient.

---

### **8. How do B-trees support efficient range queries?**

Since data in B-tree leaf nodes is stored in sorted order with linked nodes, scanning sequentially is efficient. No hashing or merging across multiple segments is required.

---

### **9. What is the trade-off of adding more indexes to a table?**

More indexes improve read speed but slow down writes because each insert/update must also update the index. Indexes also add storage overhead.

---

### **10. How does columnar storage improve compression?**

Columns contain similar values and patterns, making them ideal for techniques like dictionary encoding, run-length encoding, deltas, and bit-packing—leading to high compression ratios.

---

---

## 🟥 **Senior-Level / System Design Questions (With Answers)**

---

### **11. In what scenarios would you choose an LSM-tree over a B-tree?**

Use an LSM-tree when:

* The workload is write-heavy (event ingestion, IoT, time-series)
* Sequential writes outperform random writes (especially on SSD/HDD)
* The dataset is large and compaction overhead is acceptable
* The system runs in distributed environments (Cassandra, Bigtable, DynamoDB)

---

### **12. What are read amplification and write amplification in storage engines?**

* **Write amplification** occurs when a single write leads to multiple disk writes (common in LSM compaction and page splits in B-trees).
* **Read amplification** occurs when multiple storage locations must be checked for a single lookup (common in LSM read paths with many SSTables).

Good storage engines balance amplification against performance.

---

### **13. Why are B-trees preferred in traditional relational databases?**

B-trees provide:

* Stable low-latency lookups
* Efficient range queries
* Update-in-place semantics aligned with ACID transactions
* Predictable performance without compaction spikes

Ideal for transactional workloads requiring consistency and fast reads.

---

### **14. Why are column stores popular in analytics and data warehouse workloads?**

Column stores allow:

* Scanning billions of rows while reading only relevant columns
* Compression that reduces I/O cost significantly
* Vectorized and SIMD execution
* Predicate pushdown and skipping irrelevant blocks

These optimizations make analytical queries dramatically faster than row-based layouts.

---

### **15. How do Bloom filters help LSM-tree performance?**

Bloom filters prevent unnecessary disk reads by checking whether a key *might exist* in an SSTable before opening it. They reduce read amplification and improve read latency, especially when many SSTables exist.

---

---

## ⭐ Bonus / Thought Question (No Single Correct Answer)

### **16. If you were designing a high-frequency trading storage layer or real-time financial ledger, which storage model would you choose and why?**

Expected reasoning:

* B-tree approach if strict consistency, fast lookups, and predictable latency are required.
* LSM-tree if logging throughput is dominant and reads can be batched or cached.
* Hybrid systems (e.g., WiredTiger) may apply depending on requirements like compaction tolerance, durability, and replication model.


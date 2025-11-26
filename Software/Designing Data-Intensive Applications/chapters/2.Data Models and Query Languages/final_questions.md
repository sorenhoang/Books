

### **1. What are the main data models discussed in this chapter?**

The chapter discusses three primary data models:

* **Relational model** (tables, rows, structured schema)
* **Document model** (hierarchical JSON-like documents)
* **Graph model** (nodes and edges to represent relationships)

Each model serves different use cases depending on data structure and query needs.

---

### **2. What is the key idea behind the relational model?**

The relational model stores data in structured tables with a predefined schema. It uses normalization and joins to organize related data across multiple tables, enabling flexible querying and strong consistency guarantees.

---

### **3. What is a document model?**

A document model stores data in self-contained hierarchical structures, often as JSON or similar formats. It’s useful when data is naturally nested or when schemas evolve frequently.

---

### **4. What is a graph model used for?**

Graph models are used when relationships between data are central and highly connected, such as social networks, recommendations, identity mappings, or fraud detection networks. They efficiently support multi-hop queries.

---

### **5. What is the difference between declarative and imperative query styles?**

* **Declarative queries** state *what* data is needed (e.g., SQL).
* **Imperative queries** describe *how* to retrieve the data step by step (e.g., manual traversal or cursor loops).

### **6. When is a document database better than a relational database?**

A document DB is better when:

* Data is hierarchical or nested
* Reads often require retrieving an entire object
* Schema changes frequently
* Relationships are simple or limited
* Joins are rare or expensive

Examples: user profiles, product catalogs, configuration data.

---

### **7. Why are joins challenging in distributed systems?**

Joins typically require fetching data from multiple tables and combining it. In distributed systems, these tables may reside on different nodes, increasing **network overhead, latency, and coordination complexity**, making joins expensive or impractical at scale.

---

### **8. How does the graph model solve problems that relational or document systems struggle with?**

Graph systems make relationships first-class, so deep relationship queries (like "friends of friends who like X") become fast and natural. Traversals follow stored edges rather than computing joins or recursive lookups, reducing query cost.

---

### **9. Why are declarative languages preferred in large-scale systems?**

Declarative queries allow the database engine to choose the optimal execution plan. This abstraction makes systems easier to evolve, optimize, and distribute because changing indexes or storage formats does not require rewriting application logic.

---

### **10. What are trade-offs of the document model?**

Pros: flexibility, natural data alignment, fewer joins.
Cons: duplication risk, inconsistent schema, update anomalies, and weaker querying capabilities for relational-style queries.

---

### **11. When would you choose a graph database over a document or relational database? Give a real example.**

Choose a graph DB when relationships are dynamic, multi-hop, or central to the domain.
**Example:** Fraud detection — tracing financial transactions across accounts requires relationship path analysis and anomaly detection across subgraphs. Document and relational models struggle with such recursive queries efficiently.

---

### **12. How do triple stores differ from property graph databases?**

Triple stores use RDF triples `(subject, predicate, object)` and support semantic reasoning and inference via SPARQL. Property graphs allow richer metadata on edges and nodes and tend to optimize for application-level traversal rather than inference. Triple stores emphasize interoperability and meaning; property graphs emphasize traversal efficiency and flexibility.

---

### **13. Explain how declarative queries help with distributed query optimization.**

Declarative queries describe the desired result without specifying execution steps. This gives the query planner freedom to optimize across distributed nodes — selecting indexes, deciding join strategies, pushing filters to storage nodes, or parallelizing tasks. Imperative logic prevents such automated optimization.

---

### **14. Describe a scenario where relational and document databases coexist.**

In an e-commerce platform:

* The **relational database** stores core transactional data like orders, payment history, and inventory with strong consistency requirements.
* A **document database** stores product catalog details or user profiles, where schema flexibility and rapid iteration matter.
  This approach is known as **polyglot persistence**, aligning storage engines with workload patterns.

---

### **15. What risks arise when using schema flexibility in document databases without discipline?**

Without rules, schema drift develops: different records may contain different field names, types, or structures. This leads to unreliable queries, inconsistent application logic, difficult migrations, and operational fragility. Validation rules, versioning, and governance mitigate this.


## 🟩 **Junior-Level Questions (with Answers)**

---

### **1. What is data serialization?**

**Answer:**
Serialization is the process of converting in-memory objects into a format that can be stored or transmitted across networks. Deserialization reconstructs those objects back into usable application data.

---

### **2. What is the difference between JSON and binary formats like Protobuf or Avro?**

**Answer:**
JSON is human-readable, text-based, and easy to debug, but less space-efficient and slower to parse. Binary formats like Protobuf or Avro are compact, faster to serialize/deserialize, and support strict schemas—but require tooling to inspect data.

---

### **3. What is schema evolution?**

**Answer:**
Schema evolution refers to safely changing the structure of data over time (like adding or renaming fields) while keeping compatibility with existing applications and stored data.

---

### **4. What type of compatibility is required when new applications must still read old stored data?**

**Answer:**
Backward compatibility — newer code can read older data.

---

### **5. What’s the main difference between REST and RPC?**

**Answer:**
REST is resource-based using standard HTTP verbs, usually with JSON. RPC is function-based, using protocols like gRPC or Thrift, generally more efficient and strongly typed.

---

---

## 🟨 **Mid-Level Questions (with Answers)**

---

### **6. Why are Protobuf and Thrift better suited for internal microservice communication than JSON?**

**Answer:**
They provide compact encodings, strict schema enforcement, faster serialization/deserialization, and reliable backward/forward compatibility—making them ideal for high-throughput, low-latency internal APIs.

---

### **7. How does Avro handle schema evolution differently from Protobuf?**

**Answer:**
Avro stores no field tags and resolves schema differences dynamically using a **writer schema + reader schema**, making it ideal for streaming and logs. Protobuf relies on stable numeric field identifiers embedded in the data, making renames safe only if tag numbers remain unchanged.

---

### **8. What is message passing and why is it useful in distributed systems?**

**Answer:**
Message passing is asynchronous communication through a message broker (Kafka, RabbitMQ). It decouples producers and consumers in time and execution, improving resilience, scalability, and distributed workflow handling.

---

### **9. What problems occur if multiple services share the same database?**

**Answer:**
It creates tight coupling, makes schema evolution risky, hinders ownership boundaries, causes performance contention, and turns the database into an integration bottleneck.

---

### **10. Why does message processing require idempotency?**

**Answer:**
Because messages may be delivered more than once in "at-least-once" delivery systems. An idempotent operation ensures processing duplicates does not affect correctness (e.g., creating an invoice only once).

---

---

## 🟥 **Senior-Level / Architecture Questions (with Answers)**

---

### **11. Compare synchronous (REST/RPC) and asynchronous (messaging) communication models. When is each appropriate?**

**Answer:**

* **Synchronous (REST/gRPC):** Ideal when immediate response is required (authentication, user actions, state validation).
* **Asynchronous (Kafka/RabbitMQ):** Ideal for workflows that take time, are decoupled, or require buffering, scaling, retries, or replay (billing, event sourcing, analytics).
  A mature architecture often uses **both**, depending on workflow semantics.

---

### **12. Why is schema governance critical in event-driven or streaming architectures?**

**Answer:**
Because messages persist and may be read by many services over time. Without strict schema versioning and compatibility rules, changes can break downstream consumers, corrupt analytics pipelines, or prevent replay of historical data.

---

### **13. What are the trade-offs between using REST vs gRPC in a microservice architecture?**

**Answer:**

| REST                                  | gRPC                                |
| ------------------------------------- | ----------------------------------- |
| Human-readable, widely interoperable  | Binary, fast, strongly typed        |
| Good for external/public APIs         | Good for internal microservices     |
| Simple evolution with optional fields | Requires stricter schema versioning |
| Limited streaming support             | Full duplex streaming               |

Choice depends on interoperability needs vs performance and structure requirements.

---

### **14. Describe a scenario where shared database integration is acceptable and another where it becomes an anti-pattern.**

**Answer:**

* **Acceptable:** A monolith or tightly integrated system with a single owning team.
* **Anti-pattern:** A distributed microservice environment where multiple independent teams write to the same tables, causing ownership conflicts, schema coupling, and operational fragility.

---

### **15. How does Change Data Capture (CDC) improve dataflow design?**

**Answer:**
CDC streams database change events (instead of sharing the database) to messaging or analytical pipelines. This preserves system ownership while enabling downstream consumers to react to changes in real time — ideal for syncing operational and analytical systems or replacing shared DB patterns.

---

---

## ⭐ Final Thought Question (Open-ended)

### **16. If you are designing a global-scale event-driven architecture, how would you choose between Avro, Protobuf, or JSON Schema?**

**Ideal reasoning:**

* **Avro** for Kafka + schema registry when streaming pipelines require replay and long-term schema evolution.
* **Protobuf** for high-performance internal RPC (gRPC).
* **JSON Schema** for public REST-facing interfaces where human readability and loose compatibility are beneficial.

A hybrid approach is usually best.

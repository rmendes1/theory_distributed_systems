# Distributed Systems L1: What and Why?

### **Lecture Overview**

**Instructor**: Lindsay Cooper (UC Santa Cruz)

**Course Focus**: Distributed systems—design, challenges, and practical applications.

### **Key Topics Covered**

### 1. 💡 What *Is* a Distributed System?

- **Informal Definition** (via **Leslie Lamport**):
    
    > “A distributed system is one in which the failure of a computer you didn’t even know existed can render your own computer unusable.”
    > 
    - Emphasizes the **interdependency** and **hidden complexity** in distributed systems.
    - Failures may be outside your control or visibility, but still impact your experience.
- **More Precise Definition** (from **Martin Kleppmann**):
    
    > A distributed system is a collection of independent computers that appears to users as a single coherent system, even though individual nodes may fail independently.
    > 
    - **Key Characteristics**:
        - **Message Passing**: Nodes don’t share memory; they communicate over a network.
        - **Partial Failure**: One component can fail while the rest keep running — and the system has to deal with it.
        - **Concurrency**: Multiple nodes work in parallel, often on different tasks.
        - **Lack of Global Clock**: No shared notion of time, which complicates synchronization.
- **Design Philosophies**:
    - 🖥️ **High Performance Computing (HPC)**:
        - Treats **any failure** as catastrophic; uses **checkpointing** to periodically save state.
        - Emphasizes tight coupling, deterministic execution, often in supercomputers or clusters.
    - ☁️ **Cloud Computing**:
        - Designed to **expect failure** ("everything fails all the time" — *Werner Vogels*).
        - Emphasizes **fault tolerance**, **redundancy**, and **graceful degradation**.

---

### 2. ⚠️ Challenges in Distributed Systems

- **Uncertainty & Incomplete Knowledge**:
    - In many failure cases, you don’t know *why* a node didn't respond:
        - Was the message dropped?
        - Did the node crash?
        - Is it just slow?
        - Or is it *Byzantine* (acting arbitrarily or maliciously)?
- **Timeouts**:
    - You must set a timeout to give up on waiting — but:
        - Too short → leads to unnecessary retries.
        - Too long → leads to sluggish responsiveness.
    - Timeout values must be tuned carefully and might differ between systems.
- **Latency & Network Delays**:
    - Even in cloud data centers, delays are not negligible or consistent.
    - Latency is variable, often unbounded — you can't assume a message will be delivered "soon."
- **Lack of Global State**:
    - In a centralized system, you can observe and control all components.
    - In distributed systems, no node has a full view of the system’s state at any given time.

---

### 3. 🚀 Why Use Distributed Systems?

- **Scalability**:
    - Distribute computation and storage to handle:
        - Large user bases (e.g., social networks).
        - Big data (e.g., scientific simulations, analytics).
    - Scale horizontally — add more nodes to increase capacity.
- **Fault Tolerance**:
    - Systems like Google Search, AWS, Netflix **must keep working** even if machines fail.
    - Distributed systems replicate data and processes to **absorb failures**.
- **Performance (Throughput)**:
    - Parallelize workloads to achieve faster aggregate performance.
    - Examples:
        - MapReduce
        - Distributed databases
        - Streaming systems
- **Geographical Distribution**:
    - Systems often serve users across continents.
    - Data may need to be processed or cached closer to users (e.g., CDNs).

---

### 🧠 Key Takeaways

- Distributed systems are not defined by geography — **any system with multiple communicating processes over a network** is distributed.
- **Failure is the norm**: Systems are designed under the assumption that components will fail unpredictably.
- Core problems in the field include:
    - **Partial failure**: Must detect, isolate, and recover from failure.
    - **Lack of knowledge**: Nodes can’t see the whole picture.
    - **No guarantees**: Messages can be lost, duplicated, or delayed indefinitely.
- The course will focus on **understanding these challenges** through hands-on implementation and theory:
    - Building fault-tolerant services.
    - Exploring consensus protocols.
    - Handling asynchrony and time in distributed systems.

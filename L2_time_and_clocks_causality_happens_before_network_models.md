# Distributed Systems L2: Time and clocks, causality and happens-before, network models

## 1. Introduction - Agenda

- Time and clocks
- Lan port diagrams
- Network models
- Causality and happens-before
- State and events

## 2. Time and Clocks

### **Uses of Time in Software**

- **Scheduling:** Setting start/end times (e.g., class schedules, assignment deadlines).
- **Marking Points in Time:** Timestamps (e.g., error logs, cache expiration).
- **Measuring Durations:** Time intervals (e.g., request timeouts, class duration).

### **Types of Clocks**

1. **Time-of-Day Clocks (Wall Clocks)**
    - Reflect current real-world time (e.g., `2025-04-09 14:30:00`).
    - **Issues:** Can jump forward/backward (due to **Network Time Protocol** sync, daylight savings).
    - **Not ideal for measuring durations** due to potential inconsistencies.
2. **Monotonic Clocks**
    - Measure elapsed time since an arbitrary point (e.g., machine boot time).
    - Always increment, never jump backward.
    - **Good for measuring intervals** (e.g., timeouts).
    - **Limitation:** Values are **machine-specific**, not comparable across systems.

| Physical clocks | Points in time | Intervals/durations |
| --- | --- | --- |
| Time-of-day clocks | 😕 | 😟 |
| Monotonic clocks | 😟 | 😃 |

### **Real-World Example: Cloudflare Bug (2016)**

- Used a time-of-day clock instead of a monotonic clock, causing issues due to clock adjustments.

## **3. Physical vs. Logical Clocks**

### **Physical Clocks (Time-of-Day & Monotonic)**

- **Problem:** Even with NTP sync, clocks drift, leading to inconsistencies.
- **Solution:** Use **logical clocks** to order events rather than measure absolute time.

### **Logical Clocks**

- Focus on **event ordering** rather than exact time.
- Key question: **Did event A happen before event B?** (A → B)
    - What does this tell me about causality?
        - A could have caused B
        - B could not have caused A

## **4. Causality and Happens-Before Relation**

### **Definition**

- **Happens-Before (→):** A partial order of events in a distributed system.
    - If A and B are on the **same process**, and A occurs before B, then **A → B**.
    - If A is a **send** and B is the corresponding **receive**, then **A → B**.
    - **Transitivity:** If **A → B** and **B → C**, then **A → C**.
    
    <img src="/images/L2/L2_image1.png"/>
    

### **Lamport (Space-Time) Diagrams**

- **Processes:** Represented as vertical lines.
- **Events:** Dots on process lines (internal, send, or receive events).
- **Messages:** Arrows between processes.

### **Example: Causal Anomaly (Alice, Bob, Carol)**

- Alice sends a message to Bob.
- Bob replies rudely before Carol receives Alice’s message.
- **Result:** Carol sees Bob’s reply first (violates causality).

<img src="/images/L2/L2_image2.png"/>

## **5. Network Models**

### **Synchronous Networks**

There is an **n** such that no message takes longer than **n units of time** to be delivered

- **Fixed upper bound** on message delivery time.
- Predictable but **unrealistic** in practice.

### **Asynchronous Networks**

- **No upper bound** on message latency.
- More realistic but harder to design for (must handle unbounded delays).
- **Preferred model** for robust protocol design.

### **Partially Synchronous Networks**

- Upper bound exists but is **unknown** in advance.
- Useful for verification but not commonly used in practice.

## **6. State and Events**

### **State**

- Represents a machine’s knowledge (e.g., memory, registers).
- Determined by **all prior events** up to that point.

<img src="/images/L2/L2_image3.png"/>

### **Events**

- Can **modify state** (e.g., variable assignment, message receipt).
- **Not all events affect state** (e.g., read-only operations).
- **Challenge:** Cannot infer event history from state alone.

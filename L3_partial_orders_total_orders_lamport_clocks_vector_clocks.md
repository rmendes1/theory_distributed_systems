# Distributed Systems L3: Partial orders, total orders, Lamport clocks, vector clocks

## Agenda

1. Recap of the "happens before" relation.
2. Introduction to partial and total orders.
3. Discussion of Lamport clocks.
4. Exploration of vector clocks.

## **1. Recap: "Happens Before" Relation**

### **Definition**

The "happens before" (→) relation is defined in three ways:

1. **Same Process:** If two events occur in the same process, the earlier one happens before the later.
2. **Message Passing:** Sending a message happens before receiving it.
3. **Transitivity:** If A → B and B → C, then A → C.

### **Key Insights**

- Events such as A and E may have a direct causal path, establishing that A → E.
- Events like A and D might be **concurrent**, meaning neither happens before the other.

## **2. Partial Orders**

### **Definition**

A **partial order** is a binary relation `≤` (reads as “is related to”) over a set `S` that satisfies three key properties:

- **Reflexivity:** for all a ∈ S, a ≤ a. ***(Every element is related to itself)***
- **Anti-symmetry:** if a ≤ b and b ≤ a, then a = b. ***(This means no two different elements can be mutually related in both directions unless they are the same.)***
- **Transitivity:** if a ≤ b and b ≤ c, then a ≤ c. ***(Relationships "chain together" properly.)***

### **Why is it called a “Partial” order?**

Because **not all pairs** of elements are necessarily comparable.

In a **total order**, every pair is comparable.

But in a **partial order**, some elements might not relate in any way.

**For example:**

Given a set of events in a distributed system A, B, C:

You may know:

- `A ≤ B` (A happened before B)
- But **nothing** connects `A` and `C` — they could be **concurrent**

That makes the ordering **partial**, not total.

### **Formal Notation**

A partial order is often represented as a **partially ordered set** or **poset**:

**(S, ≤)**

Where:

- `S` is the set of elements (like events, timestamps, etc.)
- `≤` is the partial order relation defined on the set

### **Example: Set Inclusion (⊆)**

Let `S = { {}, {a}, {b}, {a,b} }`

The subset relation ⊆ defines a partial order on this set.

- `{a} ⊆ {a,b}`
- `{b} ⊆ {a,b}`
- But `{a}` and `{b}` are **not** subsets of each other ⇒ **incomparable**

So this is a **partial order**, not total.

### **"Happens Before" as a Partial Order**

- The "happens before" relation is:
    - **Transitive**
    - **Anti-symmetric**
    - **Not reflexive** → because an event doesn’t happen before itself.

Hence, it's classified as a **strict (irreflexive) partial order**.

### **Set Containment Example**

- The power set of `{a, b, c}` demonstrates a partial order using set inclusion (⊆).
- **Subset inclusion** satisfies all three partial order properties:
    - Every set is a subset of itself (reflexivity).
    - Mutual subset relationship implies equality (anti-symmetry).
    - Chain of subsets is transitive.
    
    <img src="/images/L3/L3_image1.png"/>
    

## **3. Total Orders**

### **Definition**

A **total order** is a partial order in which every pair of elements is comparable.

### **Comparison**

- **Partial Order:** Not all elements need to be comparable (e.g., concurrent events).
- **Total Order:** Every pair (a, b) must satisfy a ≤ b or b ≤ a.

### **Example**

- **Natural numbers (ℕ):** Totally ordered.
- **Set containment:** Partially ordered (some sets are incomparable).

## **4. Logical Clocks in Distributed Systems**

### **Why Logical Clocks?**

- **Physical clocks** can't accurately capture causal relationships in distributed systems.
- **Logical clocks** track event order instead of real time.

## **5. Lamport Clocks**

### **Definition**

Assigns a **timestamp** (natural number) to each event such that:

- If A → B, then `lc(A) < lc(B)`

<aside>
💡

Lamport clocks are consistent with causality.

</aside>

### **Algorithm**

1. **Initialization:** Set each process’s counter to 0. 
2. **Event Handling:**
    - Increment the counter before each event.
3. **Message Send:**
    - Increment counter and attach it to the message.
4. **Message Receive:**
    - Set counter to `max(local counter, received counter) + 1`.

### **Properties**

- **Maintains causality consistency**: A → B ⇒ `lc(A) < lc(B)`
- **Limitation:** `lc(A) < lc(B)` does **not** imply A → B.

### Why  `lc(A) < lc(B)` does **not** imply A → B?

Lamport clocks are **scalar** counters. They give us a total order of timestamps, but **not all timestamp differences represent causality** — only the ones that came through real communication.

### Consider this scenario:

<img src="/images/L3/L3_image2.png"/>

- Both events are **independent**, no messages between them.
- So **A || B** (they are concurrent).
- But still, LC(A) = 1, LC(B) = 5 ⇒ LC(A) < LC(B)

> Here, Lamport timestamps suggest A happened before B, but they are actually concurrent.
> 

So, in reality, Lamport Clocks are actually consistent with **potential** causality. That means:

- A → B ⇒ Logical clock of A < Logical clock of B ✅ (consistent with causality)
- Logical clock of A < Logical clock of B → A → B ❌ (characterizes causality)

## **6. Limitations of Lamport Clocks**

- Lamport clocks **do not fully capture causality**:
    - Two events may have different clock values but no causal connection.
- However, if `lc(A) ≥ lc(B)`, we can safely say A did **not** happen before B.
- Efficient: Only one integer needs to be attached to messages.

## **7. Vector Clocks**

### **Motivation**

To overcome Lamport clock limitations and **accurately capture causality**.

$$
A→B ↔ VC(A) < VC(B)
$$

### **Structure**

- Each process maintains a vector of integers.
- Vector size = number of processes.
- Initially, all entries = 0.

### **Algorithm**

1. **Internal Event:** Increment local process’s own position in the vector.
2. **Message Send:**
    - Increment local entry.
    - Attach entire vector to the message.
3. **Message Receive:**
    - Take **element-wise max** of local and received vectors.  `max(local, received)`
    - Increment local process’s entry.

### **Properties**

- `vc(A) < vc(B)` ⇔ A → B (true implication both ways).
- More **informative** than Lamport clocks:
    - Can tell which process has seen which events.
    - Can detect concurrency accurately.

## **8. Concluding Notes**

- **Partial vs Total Orders:** Key for modeling event relationships.
- **Lamport Clocks:** Simple and efficient, but don’t fully reflect causality.
- **Vector Clocks:** Stronger, more accurate representation of causality.

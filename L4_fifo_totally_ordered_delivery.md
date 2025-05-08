# Distributed Systems  L4: vector clocks, FIFO/causal/totally-ordered delivery

 🗓️ **Agenda Overview**

- Recap of Lamport clocks
- Introduction to Vector clocks
- Executions and properties
- Message delivery guarantees:
    - FIFO
    - Causal
    - Totally Ordered Delivery

## 🧠 **Recap: Lamport Clocks and Causality**

- If event **a happens before b**, then **LC(a) < LC(b)**.
- The reverse **is not always true**; **LC(a) < LC(b)** does **not** imply **a → b**.
- Lamport clocks help establish a partial ordering but not causality.

## ⏰ **Vector Clocks: Concepts and Usage**

- **Purpose**: Capture causal relationships among events in distributed systems.
- **Structure**: Each process maintains a vector of clock values (one per process).
- **Initialization**: All vector values start at 0.

### 🔁 **Update Rules:**

1. **Local event** → Increment own entry in vector.
2. **Sending message** → Send current vector with message.
3. **Receiving message** → Take **pointwise max** between own and received vector, then increment own entry.

### 🔍 **Example (Alice, Bob, Carol)**:

- Bob receives a message → combines clocks: `max([0,1,0], [0,1,1]) = [0,1,1]`
- Then increments own entry → `[0,2,1]`

## 🔄 **Causal History and Future**

- **Causal History**: All events that happened-before a given event.
- **Causal Future**: All events that happen-after.
- Vector clocks:
    - **History** → strictly smaller
    - **Future** → greater than or equal

## 📦 **Protocols and Valid Executions**

- Different valid runs of the protocol may exist.
- Logical ordering of events matters more than real-time ordering.
- Diagrams can help visualize correct/incorrect protocol behavior.

## 📬 **Message Delivery Guarantees**

### 1. **FIFO (First-In-First-Out) Delivery**

Ensures that **messages sent by a single sender** to a single receiver are **delivered in the order they were sent**.

**🧭 Example:**

If Alice sends two messages to Bob:

- Message 1: "Hello"
- Message 2: "How are you?"

Then **FIFO delivery guarantees** that Bob will receive "Hello" **before** "How are you?".

> ❗ FIFO does not impose any order between messages from different senders.
> 

**🛠 How is FIFO Delivery Implemented?**

✅ 1. **Sequence Numbers**

- Each sender attaches a **monotonically increasing sequence number** to each message.
- The receiver keeps track of the **next expected sequence number** from each sender.
- If a message arrives **out of order**, it’s **buffered** until earlier messages are received.

✅ 2. **Message Queuing**

- Receivers maintain per-sender queues.
- Messages are stored until they can be **delivered in order**.

**⚠ Challenges in FIFO Delivery**

| Challenge | Description |
| --- | --- |
| **Lost Messages** | If a message is lost, subsequent ones must be held indefinitely unless there's a recovery mechanism. |
| **Delays** | A delayed message can block delivery of later messages, even if they’ve arrived. |
| **Buffering** | Requires memory to store out-of-order messages until earlier ones arrive. |
| **Reliable Delivery Required** | FIFO depends on **reliable delivery**, otherwise the receiver might wait forever for a missing message. |

**🔍 Example Scenario:**

Alice → Bob

Messages: M1, M2, M3

- M2 and M3 arrive before M1.
- Bob buffers M2 and M3.
- Once M1 arrives, Bob delivers M1, then M2, then M3.

**💡 Is TCP FIFO?**

Yes!

**TCP (Transmission Control Protocol)** ensures reliable and **in-order (FIFO)** delivery of bytes between two endpoints — so if you’re using TCP sockets, **you already get FIFO** for free.

However, if you’re using **UDP** or a **custom protocol**, you might have to enforce FIFO delivery manually.

### 2. **Causal Delivery**

**Causal Delivery** ensures that **messages are delivered in an order that respects causal relationships**.

That is, if **message A could have influenced message B**, then every process must **deliver A before B**.

**🔗 Causal Relationship: “Happens-Before”**

Causality between messages is based on the **happens-before** relation (`→`), which includes:

1. Events in the same process:
    
    If event A occurs before event B in the same process, then A → B.
    
2. Message sending and receiving:
    
    If a process sends message A before receiving message B, then A → B.
    

**🧭 Example:**

Alice sends message **M1** to Bob.

Bob, after receiving **M1**, sends **M2** to Carol.

Then, **M1 → M2** (M1 causally precedes M2).

So **Carol must deliver M1 before M2** — even if M2 arrives first.

**🛠 How is Causal Delivery Implemented?**

### ✅ 1. **Vector Clocks**

- Each process maintains a **vector clock**.
- Each message is sent with the sender’s current vector.
- Upon receiving a message, the receiver checks:
    - **Have all causally prior messages been delivered?**
    - If not, the message is **buffered** until dependencies are satisfied.

### ✅ 2. **Delivery Conditions**

To deliver a message `m` from process `P` with vector clock `V_m`, the receiver ensures:

- For all `i ≠ P`, the receiver’s clock `V_r[i] ≥ V_m[i]`
- For `P`, it expects `V_r[P] + 1 == V_m[P]`

### ⚠ Challenges in Causal Delivery

| Challenge | Description |
| --- | --- |
| **Overhead** | Vector clocks grow with the number of processes (size = N). |
| **Buffering** | Messages might be buffered for a long time if causal dependencies are delayed. |
| **Concurrency Detection** | Must distinguish between concurrent and causally-related events. |
| **Reliable Delivery Needed** | Just like FIFO, assumes messages will eventually arrive. |

### 🤔 Why Causal Delivery Matters

- Prevents **causal inconsistencies** (e.g., reading a reply before the question was "seen").
- Useful in **collaborative apps**, **chat systems**, **replicated databases**, etc.

### 3. **Totally Ordered Delivery**

- All processes must **agree on the order** of message deliveries.
- Causality is not required, but global consistency is.
- Violations occur when different processes observe **different message orders**.

### 🔑 **Example Violation**:

- Clients write to a key-value store.
- Due to inconsistent delivery order at replicas, state diverges.

## 🧪 **Practical Considerations**

- FIFO alone is not enough for causal or total order.
- Using vector clocks for causal delivery can be costly.
- Total ordering is **hard to enforce** and discussed further in later lectures.

## 🧾 **Alternative FIFO Implementation Idea**

- Use **timeouts and ACKs**:
    - Each message has a timer.
    - Sender waits for ACK before sending the next message.
- Drawbacks:
    - Can lead to slow performance.
    - Risk of lost ACKs → duplication or indefinite waits.

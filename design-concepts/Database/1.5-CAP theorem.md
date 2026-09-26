# CAP Theorem

## 1. What problem does CAP theorem solve?

Imagine you have a distributed database:

```text
              Application
                   |
          +--------+--------+
          |                 |
       DB Node A          DB Node B
        (Primary)          (Replica)
```

Both nodes contain the same data.

Now suppose the network between them breaks:

```text
              Application
              /         \
             /           \
        DB Node A       DB Node B
             XXXXXXXXXXXXX
              Network
              partition
```

The two database nodes can no longer communicate.

Now a difficult question appears:

> Should both nodes continue accepting requests, or should one/both nodes stop serving requests to guarantee that clients always see consistent data?

**CAP theorem is about this exact situation.**

---

# 2. CAP stands for

CAP means:

| Letter | Meaning             | Simple meaning                                          |
| ------ | ------------------- | ------------------------------------------------------- |
| **C**  | Consistency         | Every read gets the latest valid value                  |
| **A**  | Availability        | Every request gets a response                           |
| **P**  | Partition Tolerance | System continues despite network failures between nodes |

The famous statement is:

> A distributed system cannot simultaneously guarantee **Consistency, Availability, and Partition Tolerance** when a network partition occurs.

But there's an important nuance:

### CAP doesn't really mean "pick any 2 out of 3."

In a distributed system, **network partitions are considered unavoidable**.

Therefore, when a partition occurs, the real choice is:

```text
             Network Partition
                    |
             +------+------+
             |             |
           CP             AP
             |             |
      Consistency       Availability
      over service      over consistency
```

You generally choose whether to sacrifice **Availability** or **Consistency** during the partition.

---

# 3. C — Consistency

CAP's **Consistency** has a very specific meaning.

It means:

> After a successful write, every subsequent read sees that latest value.

For example:

```text
Initial:
User balance = $100

WRITE balance = $50
```

After the write succeeds:

```text
READ → $50
READ → $50
READ → $50
```

You shouldn't get:

```text
READ → $100   ❌
READ → $50    ✅
READ → $100   ❌
```

However, don't confuse CAP consistency with **database transaction consistency in ACID**.

They're different concepts.

### ACID Consistency

Means:

> A transaction moves the database from one valid state to another valid state according to its constraints.

### CAP Consistency

Means:

> All nodes/clients see a consistent/latest value in the presence of distributed operations.

This distinction is **very important for system design interviews**.

---

# 4. A — Availability

Availability means:

> Every request receives a response, without the system rejecting the request simply because another node is unreachable.

For example:

```text
Client
  |
  v
DB Node B

Network partition exists
DB Node A ←X→ DB Node B
```

If Node B is still accepting requests:

```text
WRITE → success
READ  → response
```

that's availability.

But the response may not necessarily contain the newest data.

---

# 5. P — Partition Tolerance

A **network partition** occurs when distributed nodes cannot communicate.

For example:

```text
       Network
          |
     +----+----+
     |         |
   Node A    Node B
     |         |
     +----X----+
       broken
     connection
```

Node A and Node B are both alive.

But:

```text
Node A  ──X──  Node B
```

They can't communicate.

Partition tolerance means:

> The system continues operating despite this communication failure.

This is extremely important because network failures are unavoidable in distributed systems.

Examples:

* Network cable failure
* Switch failure
* Router failure
* DNS problems
* Packet loss
* Availability-zone failure
* Data-center connectivity failure

---

# 6. The famous CAP scenario

Let's make this concrete.

Suppose we have two replicas:

```text
                 Application
                  /       \
                 /         \
             Node A       Node B
              DB            DB

              balance = $100
```

Now the network breaks:

```text
             Node A   X   Node B
               |             |
           balance=100    balance=100
```

A client sends:

```text
WRITE balance = $50
```

But suppose the request reaches Node A.

Node A now has:

```text
Node A → $50
Node B → $100
```

Node A cannot tell Node B because of the network partition.

Now another client asks Node B:

```text
READ balance
```

What should Node B return?

---

# 7. Option 1 — Choose Consistency

Node B knows that it cannot communicate with Node A.

It therefore says:

> "I cannot guarantee that my value is the latest one."

So it refuses/delays the request.

```text
Client
  |
  v
Node B
  |
  X
  |
Cannot verify latest data
```

You preserve:

```text
Consistency ✅
Partition tolerance ✅
Availability ❌
```

This is a **CP system**.

---

# 8. Option 2 — Choose Availability

Instead, Node B says:

> "I'll continue responding even though I can't communicate with Node A."

So:

```text
Client → Node B
           |
           v
       balance = $100
```

The client receives:

```text
$100
```

even though Node A already has:

```text
$50
```

Now:

```text
Availability ✅
Partition tolerance ✅
Consistency ❌
```

This is an **AP system**.

Later, when the network recovers:

```text
Node A ←────────→ Node B
```

the system can reconcile the differences.

---

# 9. CP vs AP

This is the heart of CAP.

### CP — Consistency + Partition Tolerance

During a network partition:

```text
Consistency
     +
Partition tolerance
     |
     v
Sacrifice Availability
```

The system may reject or delay requests.

Example conceptual behavior:

```text
WRITE → accepted by quorum
READ  → requires quorum

If quorum unavailable:
       ↓
Request fails
```

The priority is:

> "I'd rather return an error than return potentially stale/wrong data."

---

### AP — Availability + Partition Tolerance

During a network partition:

```text
Availability
      +
Partition tolerance
      |
      v
Sacrifice immediate consistency
```

The system continues serving requests.

```text
Node A → accepts writes
Node B → accepts writes

Network partition
      ↓
Nodes may temporarily disagree
      ↓
Network recovers
      ↓
Conflict reconciliation
```

The priority is:

> "I'd rather keep the system running and reconcile differences later."

---

# 10. What about CA?

You will often see:

```text
CA = Consistency + Availability
CP = Consistency + Partition tolerance
AP = Availability + Partition tolerance
```

But there's an important catch.

A true distributed system cannot simply say:

> "I'll choose CA and ignore partition tolerance."

Because if a network partition happens, you **must** decide what happens.

For example:

```text
Node A ←X→ Node B
```

You can't guarantee both:

```text
C = Consistency
A = Availability
```

while also tolerating that partition.

Therefore:

### CA is mostly relevant to systems where partitioning isn't part of the distributed-system model.

For modern distributed databases, the interesting discussion is usually:

```text
             Partition
                 |
           +-----+-----+
           |           |
          CP          AP
```

---

# 11. Real-world example — Banking

Imagine:

```text
Bank DB
   |
   +--- Mumbai
   |
   +--- Dubai
```

Suppose network connectivity between them breaks.

A customer in Dubai tries:

```text
Transfer ₹10,000
```

If the system is designed to prioritize consistency, it may say:

```text
Transaction cannot be completed
Please retry
```

instead of risking:

```text
Dubai DB → ₹10,000 transferred

Mumbai DB → doesn't know about transfer
```

Why?

Because financial systems generally have operations where stale/conflicting state can be very problematic.

This is a classic situation where **strong consistency can be prioritized over availability during a partition**.

---

# 12. Example — Social media likes

Now imagine:

```text
Post
Likes = 1000
```

A user clicks:

```text
LIKE ❤️
```

But there's a network partition.

It may be acceptable for one replica to temporarily show:

```text
1001 likes
```

while another shows:

```text
1000 likes
```

Later:

```text
Network recovers
       ↓
Replicas synchronize
       ↓
Likes = 1001
```

For this type of workload, continuing to serve requests can be more important than immediate global consistency.

That's the kind of scenario associated with **AP-style behavior**.

---

# 13. CAP and replication

This connects directly to the previous topic you studied: **replication**.

Suppose:

```text
             Primary
                |
          +-----+-----+
          |           |
       Replica A   Replica B
```

With synchronous replication:

```text
WRITE
  |
  v
Primary
  |
  +----> Replica A
  |
  +----> Replica B
  |
  v
ACK
```

The system may require replicas to acknowledge before returning success.

That can improve consistency, but during a network failure:

```text
Primary ──X── Replica
```

writes may have to stop.

That's the CAP trade-off appearing in practice.

---

# 14. Quorum and CAP

A very important concept here is **quorum**.

Suppose you have:

```text
3 replicas

A
B
C
```

You might require:

```text
Write quorum = 2
Read quorum  = 2
```

So a write:

```text
WRITE
  |
  +--> A ✅
  +--> B ✅
  +--> C ❌

2 acknowledgements
      ↓
WRITE SUCCESS
```

If only one node is reachable:

```text
A ←X→ B
     X
     C
```

you don't have enough nodes for quorum.

A CP-oriented system may therefore return:

```text
ERROR / unavailable
```

rather than accepting potentially inconsistent data.

This is one mechanism through which distributed databases implement consistency guarantees.

---

# 15. CAP vs PACELC

Once you understand CAP, you'll eventually encounter **PACELC**.

CAP talks about:

> What happens **when a partition occurs?**

PACELC extends that:

```text
If Partition:
    choose Availability or Consistency

Else:
    choose Latency or Consistency
```

Hence:

```text
P → A/C
E → L/C
```

Where:

* **P** = Partition
* **A** = Availability
* **C** = Consistency
* **E** = Else
* **L** = Latency

This is a useful advanced system-design concept, but you don't need to go deep into it yet.

---

# 16. Very important misconception

You might hear someone say:

> "MongoDB is AP."

or

> "PostgreSQL is CA."

Be careful with statements like these.

Modern databases can have **different consistency and availability behaviors depending on configuration, replication topology, failure mode, read/write settings, and guarantees being discussed**.

So CAP isn't really a permanent sticker you put on a database.

Instead, ask:

> **What consistency and availability guarantees does this system provide under a network partition?**

That's the more accurate system-design question.

---

# 17. CAP in one diagram

Keep this mental model:

```text
                 Distributed System
                         |
                  Network partition
                         |
                 +-------+-------+
                 |               |
                 |               |
                CP              AP
                 |               |
        Consistency first   Availability first
                 |               |
        Requests may fail   Requests continue
                 |               |
        No stale/conflicting Temporary inconsistency
        data during partition   possible
                 |               |
                 +-------+-------+
                         |
                 Partition Tolerance
```

The key thing is:

> **Partition tolerance isn't really optional in a distributed system. The practical CAP trade-off is what you sacrifice when a partition occurs: consistency or availability.**

---

# 18. CAP vs the topics you've already learned

Your roadmap is actually lining up nicely:

```text
Replication
    ↓
Multiple copies of data
    ↓
Sharding
    ↓
Data distributed across nodes
    ↓
CAP theorem
    ↓
What happens when those nodes
can't communicate?
    ↓
Consistency models
    ↓
How much consistency do we actually need?
    ↓
Database scaling
    ↓
How do we handle increasing load?
```

So the **next topic, #8 Consistency Models**, follows directly from CAP.

And that's where we'll get into things like:

```text
Strong consistency
Eventual consistency
Read-after-write consistency
Monotonic reads
Causal consistency
```

Those concepts make CAP much more practical for system design.

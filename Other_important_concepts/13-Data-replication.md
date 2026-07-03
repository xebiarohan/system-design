Replication is the second major concept in distributed storage after sharding.

A useful way to remember the difference is:

* **Sharding** answers: *"How do we split the data?"*
* **Replication** answers: *"How do we make copies of the data?"*

You almost always use **both** in large-scale systems.

---

# What is Replication?

Replication means storing **multiple copies of the same data** on different servers.

Without replication:

```text
          Users DB
      +-------------+
      | User Data   |
      +-------------+
```

If this server crashes:

* ❌ Data is unavailable
* ❌ Users cannot log in
* ❌ Application may go down

With replication:

```text
             Primary
          +-----------+
          | User Data |
          +-----------+
           /        \
          /          \
+-----------+     +-----------+
| Replica 1 |     | Replica 2 |
+-----------+     +-----------+
```

Now if one server fails, another copy still exists.

---

# Why do we need Replication?

Replication provides several benefits.

## 1. High Availability

If the primary database crashes:

```text
Primary ❌
```

The system can promote a replica:

```text
Replica 1
↓

New Primary
```

The application continues working.

---

## 2. Fault Tolerance

Imagine a hard disk fails.

Without replication:

```text
Disk Failure

↓

Data Lost
```

With replication:

```text
Disk Failure

↓

Read data from Replica
```

No data is lost.

---

## 3. Better Read Performance

Suppose your application receives:

```text
1,000,000 reads/sec
10,000 writes/sec
```

A single database cannot handle all reads.

Instead:

```text
                Primary
                   |
        -----------------------
        |         |          |
    Replica1  Replica2  Replica3
```

Writes go to the primary.

Reads are distributed across replicas.

This dramatically improves scalability.

---

# Replication Models

There are two major replication strategies.

1. **Synchronous Replication**
2. **Asynchronous Replication**

---

# 1. Synchronous Replication

Every write must be copied to replicas **before** it is considered successful.

Suppose a user changes their password.

The application sends:

```text
Update Password
```

Flow:

```text
Client
   |
Primary
   |
Replica 1
   |
Replica 2
   |
Success returned
```

The client waits until every required replica confirms the write.

---

Example

Change password:

```text
Old:
123456

New:
abc123
```

Primary updates.

Replica updates.

Only then:

```text
200 OK
```

is returned.

---

## Advantages

### Strong Consistency

Every replica has the same data.

No stale reads.

Example:

```text
Write:
Balance = $500
```

Immediately afterward:

```text
Read from Replica

↓

$500
```

Always correct.

---

### Better for Critical Data

Examples:

* Banking
* Payments
* Inventory management

You cannot afford replicas having different values.

---

## Disadvantages

### Slower Writes

The client waits for replicas.

Suppose:

```text
Primary

5 ms
```

Replica:

```text
15 ms
```

Total write time:

```text
20 ms
```

Latency increases.

---

### Lower Availability

If a replica is unreachable:

```text
Primary

↓

Replica ❌
```

The write may fail because the system requires confirmation.

---

# 2. Asynchronous Replication

Here the primary immediately responds after writing locally.

Replication happens later.

Flow:

```text
Client

↓

Primary

↓

200 OK

↓

Replica updates later
```

The user gets a faster response.

---

Timeline

```text
0 ms

Write Primary

5 ms

Client Success

20 ms

Replica Updated
```

---

## Advantages

### Fast Writes

No waiting for replicas.

Much lower latency.

---

### Better Availability

Even if replicas are offline:

```text
Primary

↓

Success
```

Writes continue.

Replicas catch up later.

---

## Disadvantages

### Replica Lag

There is a delay before replicas receive updates.

Example

Primary:

```text
Balance

$500
```

Replica:

```text
Still

$400
```

For a few seconds.

---

This causes **eventual consistency**.

---

# Eventual Consistency

Suppose:

```text
Primary

Name = Alice
```

Replica still has:

```text
Name = Alicia
```

A few seconds later:

```text
Replica

↓

Alice
```

Eventually all replicas agree.

This is called **eventual consistency**.

---

# Primary-Replica (Leader-Follower) Replication

This is the most common replication architecture.

```text
               Primary
          (Leader Database)
                 |
       -----------------------
       |          |          |
   Replica1   Replica2   Replica3
```

Rules:

* All writes go to the primary.
* Replicas copy data from the primary.
* Reads can come from replicas (depending on consistency needs).

Example:

```text
INSERT User
```

Application:

```text
↓

Primary
```

Then:

```text
Primary

↓

Replicas
```

---

## Read and Write Flow

User logs in.

Application needs profile.

```text
Application

↓

Replica
```

Fast read.

User updates email.

```text
Application

↓

Primary
```

Write happens only on the primary.

---

# Multi-Primary (Multi-Leader) Replication

Instead of one primary:

```text
Primary A

Primary B

Primary C
```

Each server accepts writes.

Example:

Europe:

```text
Primary EU
```

America:

```text
Primary US
```

Asia:

```text
Primary Asia
```

Each region writes locally and later synchronizes changes.

---

## Advantages

* Low write latency worldwide
* No single write bottleneck
* Better resilience if one region is unavailable

---

## Disadvantages

### Write Conflicts

Suppose two users edit the same profile at nearly the same time.

Europe:

```text
Name = John
```

America:

```text
Name = Johnny
```

When synchronization happens, which value should win?

Conflict resolution becomes necessary.

Common strategies include:

* Last write wins (based on timestamps)
* Version vectors
* Manual conflict resolution
* Application-specific merge logic

---

# Quorum-Based Replication

Many distributed databases (such as those inspired by Dynamo) use a quorum approach.

Assume:

```text
3 replicas
```

We define:

* **N** = total replicas = 3
* **W** = replicas that must acknowledge a write
* **R** = replicas that must respond to a read

Example:

```text
N = 3
W = 2
R = 2
```

A write succeeds after **2 of 3** replicas confirm.

A read consults **2 of 3** replicas and returns the newest value.

A common rule is:

```text
R + W > N
```

This increases the likelihood that reads observe the latest successful write.

---

# Failover

Suppose the primary crashes.

Before:

```text
Primary

↓

Replica1

Replica2
```

Primary fails:

```text
Primary ❌
```

A replica is promoted:

```text
Replica1

↓

New Primary
```

Applications reconnect to the new primary.

This process is called **failover**.

---

# Sharding vs Replication

This is a very common interview question.

| Feature              | Sharding                           | Replication                                |
| -------------------- | ---------------------------------- | ------------------------------------------ |
| Purpose              | Split data across servers          | Create copies of the same data             |
| Goal                 | Scale storage and write throughput | Improve availability and read throughput   |
| Data on each server  | Different                          | Same                                       |
| Handles more writes? | ✅ Yes                              | ❌ Not directly (in leader-follower setups) |
| Handles failures?    | Partially                          | ✅ Yes                                      |

---

# Using Both Together

Real systems combine both techniques.

Imagine 1 billion users.

First, split users into shards:

```text
Shard 1
Shard 2
Shard 3
```

Then replicate each shard:

```text
          Shard 1
            |
      --------------
      |            |
  Replica A    Replica B

          Shard 2
            |
      --------------
      |            |
  Replica A    Replica B

          Shard 3
            |
      --------------
      |            |
  Replica A    Replica B
```

This architecture gives you:

* **Sharding** for horizontal scaling of storage and writes.
* **Replication** for high availability, fault tolerance, and scalable reads.

---

# Interview Tips

When discussing replication in a system design interview, interviewers often expect you to explain these trade-offs:

* **Synchronous replication** → Higher consistency, slower writes.
* **Asynchronous replication** → Faster writes, eventual consistency due to replica lag.
* **Leader-follower replication** → Simple and widely used, but the leader can become a write bottleneck.
* **Multi-leader replication** → Better for geographically distributed writes, but introduces conflict resolution challenges.
* **Quorum replication** → Balances consistency and availability through configurable read/write acknowledgments.

Understanding **when** to choose each strategy is usually more important than memorizing their definitions.

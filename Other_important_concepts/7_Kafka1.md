# 6. Kafka

* Kafka is a distributed event streaming platform
* Used for high-throughput, fault-tolerant, asynchronous communication between systems
* Producers write messages to Kafka
* Consumers read messages from Kafka
* Kafka acts as a durable event store

```text
Producer --> Kafka --> Consumer
```

### Why Kafka?

* Decouples services
* Handles very high traffic
* Supports replaying old events
* Fault tolerant through replication
* Horizontal scalability

### Common Use Cases

* Order processing
* Payment processing
* Event-driven architecture
* Log aggregation
* Real-time analytics
* Microservice communication

### Kafka Components

* Producer
* Topic
* Partition
* Broker
* Consumer
* Consumer Group
* ZooKeeper (older versions)
* KRaft Controller (newer versions)

---

# 7. Kafka Topic

* A Topic is a logical category or stream of messages/events
* Producers publish messages to topics
* Consumers subscribe to topics

Example:

```text
orders
payments
shipments
notifications
```

### Topic Characteristics

* Topics are append-only logs
* Messages are written sequentially
* Messages are retained for configurable time
* Multiple consumers can read the same message

### Topic Retention

Messages are not deleted immediately after consumption.

Example:

```text
Retention = 7 days
```

Even if consumed:

```text
Order Created
Payment Done
Order Shipped
```

remain available for replay until retention expires.

### Topic Replication

* Topics can be replicated across brokers
* Provides fault tolerance
* One replica acts as Leader
* Others act as Followers

```text
Partition 0

Leader    -> Broker 1
Follower  -> Broker 2
Follower  -> Broker 3
```

---

# 8. Kafka Partition

* A Topic is divided into one or more Partitions
* Partition is the unit of parallelism in Kafka
* Each partition is an ordered append-only log

Example:

```text
Topic: orders

P0
P1
P2
```

### Why Partitions?

#### Scalability

Instead of storing all data on one machine:

```text
orders
 |
 +-- P0
 +-- P1
 +-- P2
```

data can be distributed across multiple brokers.

#### Parallel Processing

Multiple consumers can process different partitions simultaneously.

### Message Ordering

Kafka guarantees ordering only within a partition.

```text
P0

Offset 0 -> Order Created
Offset 1 -> Payment Done
Offset 2 -> Shipped
```

Consumer always reads:

```text
Order Created
Payment Done
Shipped
```

### Offsets

* Every message in a partition has a unique offset
* Offsets are sequential numbers

Example:

```text
P0

Offset 0
Offset 1
Offset 2
Offset 3
```

Important:

```text
Offsets are unique within a partition
NOT across topic
```

### Partition Selection

#### Round Robin

```text
Msg1 -> P0
Msg2 -> P1
Msg3 -> P2
```

Used for even distribution.

#### Key-Based Partitioning

```java
key = customerId
```

```text
customer-101 -> P1
customer-101 -> P1
customer-101 -> P1
```

Ensures ordering for a particular key.

### Partition Count and Parallelism

```text
Partitions = Maximum Parallelism
```

Example:

```text
12 Partitions
```

Maximum:

```text
12 active consumers
```

inside a consumer group.

---

# 9. Consumer Group

* A Consumer Group is a collection of consumers working together
* Kafka distributes partitions among consumers of the same group
* Each partition is consumed by only one consumer within a group

Example:

```text
orders topic

Partitions:
P0 P1 P2 P3
```

Consumer Group:

```text
C1
C2
```

Assignment:

```text
C1 -> P0 P1
C2 -> P2 P3
```

### Why Consumer Groups?

Without Consumer Groups:

```text
All consumers read all messages
```

With Consumer Groups:

```text
Work is distributed
```

allowing horizontal scaling.

---

# 10. Consumer Group Offsets

Each consumer group maintains its own offsets.

Example:

```text
P0

Offset 0
Offset 1
Offset 2
Offset 3
```

Consumer Groups:

```text
order-service
analytics-service
```

Current position:

```text
order-service     -> Offset 3
analytics-service -> Offset 1
```

Both consume independently.

### Benefit

Multiple applications can process same data without affecting each other.

---

# 11. Consumer to Partition Assignment

### Case 1

```text
Partitions = 6
Consumers  = 4
```

Possible assignment:

```text
C1 -> P0 P1
C2 -> P2 P3
C3 -> P4
C4 -> P5
```

### Case 2

```text
Partitions = 6
Consumers = 5
```

Possible assignment:

```text
C1 -> P0
C2 -> P1
C3 -> P2
C4 -> P3
C5 -> P4 P5
```

### Case 3

```text
Partitions = 6
Consumers = 6
```

```text
C1 -> P0
C2 -> P1
C3 -> P2
C4 -> P3
C5 -> P4
C6 -> P5
```

Maximum utilization.

### Case 4

```text
Partitions = 6
Consumers = 7
```

```text
C1 -> P0
C2 -> P1
C3 -> P2
C4 -> P3
C5 -> P4
C6 -> P5
C7 -> Idle
```

One consumer remains idle.

---

# 12. Rebalancing

* Rebalancing occurs when consumers join or leave a group
* Kafka redistributes partitions automatically

### New Consumer Joins

Before:

```text
C1 -> P0 P1
C2 -> P2 P3
```

Consumer C3 joins.

After:

```text
C1 -> P0
C2 -> P1
C3 -> P2 P3
```

### Consumer Crash

Before:

```text
C1 -> P0 P1
C2 -> P2 P3
```

C2 crashes.

After rebalance:

```text
C1 -> P0 P1 P2 P3
```

### Drawback

During rebalance:

```text
Consumption temporarily pauses
```

Modern Kafka uses cooperative rebalancing to reduce disruption.

---

# Interview One-Liners

### What is a Kafka Topic?

> A topic is a logical stream of events where producers publish messages and consumers subscribe to read them.

### What is a Partition?

> A partition is an ordered append-only log inside a topic and is the unit of scalability and parallelism in Kafka.

### What is a Consumer Group?

> A consumer group is a set of consumers that share the work of consuming partitions from a topic.

### How many consumers can actively consume in a consumer group?

> Maximum active consumers = Number of partitions.

### Does Kafka guarantee ordering?

> Yes, but only within a partition.

### Can two consumers consume the same partition in the same consumer group?

> No. A partition can be assigned to only one consumer within a consumer group at a time.

### Why use message keys?

> To ensure all events with the same key are routed to the same partition and processed in order.

### Delete After Consumption in Kafka
    Kafka does not remove messages after consumption.
    Consumers track offsets, not message ownership.
    Messages remain available for replay.
    Deletion is controlled by:
    Time-based retention
    Size-based retention
    Log compaction
    If true delete-after-read behavior is required, use a message queue (RabbitMQ, SQS, ActiveMQ) instead of Kafka.
    Kafka is an event streaming platform, not a traditional queue.
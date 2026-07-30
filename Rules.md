In a system design interview, the interviewer is usually evaluating **how you think**, **how you make trade-offs**, and **how well you can communicate a scalable solution**. The biggest mistake candidates make is jumping straight into architecture without first understanding the problem.

A structured approach helps you cover all important aspects. A common framework is:

> **Requirements → Capacity Estimation → APIs → High-Level Design → Database → Detailed Components → Scaling → Bottlenecks → Trade-offs**

Below is a detailed walkthrough.

---

# 1. Clarify the Problem (2–5 minutes)

Never assume requirements.

Ask questions like:

* Who are the users?
* What are the major use cases?
* Is it read-heavy or write-heavy?
* Is this global or regional?
* What scale are we targeting?
* Is real-time communication required?
* Any latency requirements?

Example:

If asked:

> Design Instagram

Ask:

* Should we support posting images?
* Videos?
* Stories?
* Comments?
* Likes?
* Search?
* Notifications?
* Messaging?

The interviewer usually narrows the scope.

---

# 2. Functional Requirements

These describe **what the system should do.**

Example (Design Twitter)

Functional Requirements:

* User registration/login
* Post tweets
* Follow/unfollow users
* View news feed
* Like tweets
* Retweet
* Search users
* Notifications

Example (URL Shortener)

Functional Requirements:

* Shorten URL
* Redirect shortened URL
* Custom alias
* Analytics
* Expiration

---

# 3. Non-Functional Requirements

These describe **how the system should behave.**

Mention things like:

### Scalability

Can support millions of users.

Example:

* 10 million daily active users
* 100K requests/sec

---

### Availability

System should stay online.

Example:

99.99% uptime

---

### Reliability

No data loss.

---

### Low Latency

Example:

Feed loading should happen in

< 200 ms

---

### Durability

Once data is stored, it shouldn't disappear.

---

### Consistency

Do users always see latest data?

Or

Eventual consistency is acceptable?

---

### Security

* Authentication
* Authorization
* Encryption
* Rate limiting

---

### Fault Tolerance

If one server crashes

System should continue working.

---

### Maintainability

Easy to add new features.

---

### Observability

* Logging
* Monitoring
* Metrics
* Alerts

---

# 4. Capacity Estimation

Interviewers love rough calculations.

Estimate:

## Users

Example:

10 million DAU

---

## Requests

Example

Each user refreshes feed

20 times/day

Total

200 million requests/day

≈2300 requests/sec

Peak

5×

≈12000 RPS

---

## Storage

Suppose

One image = 2 MB

1 million uploads/day

Storage/day

2 TB

Storage/year

730 TB

---

## Bandwidth

Estimate upload/download traffic.

---

No need for perfect numbers.

Reasonable assumptions matter.

---

# 5. High-Level Architecture

Draw major components.

Example

```
Clients
    |
Load Balancer
    |
API Gateway
    |
Application Servers
    |
---------------------------
| User Service            |
| Feed Service            |
| Media Service           |
| Notification Service    |
---------------------------
       |
Databases
       |
Cache
       |
Message Queue
       |
Object Storage
```

Explain the responsibility of each service.

---

# 6. API Design

Design REST/gRPC APIs.

Example

Create Tweet

```
POST /tweets
```

Response

```
{
"id":123
}
```

Fetch Feed

```
GET /feed?page=2
```

Like Tweet

```
POST /tweets/{id}/like
```

---

# 7. Database Design

Discuss:

### Entities

Example

Twitter

```
User

user_id
name
email

Tweet

tweet_id
user_id
text
timestamp

Followers

user_id
follower_id

Likes

user_id
tweet_id
```

---

### SQL vs NoSQL

Explain why.

SQL

* transactions
* joins

NoSQL

* scalability
* flexible schema
* huge traffic

---

### Indexing

Example

Index

```
user_id
tweet_id
timestamp
```

---

### Partitioning

Shard by

```
User ID
```

or

```
Region
```

---

# 8. Data Flow

Explain step by step.

Example

Uploading photo

```
Client

↓

API

↓

Authentication

↓

Media Service

↓

Object Storage

↓

Metadata DB

↓

Message Queue

↓

Notification Service
```

Interviewers like hearing the sequence.

---

# 9. Caching Strategy

Mention:

Redis

Memcached

What to cache?

* User profiles
* Feed
* Trending topics
* Sessions

Discuss

Cache eviction

TTL

Cache invalidation

Cache-aside pattern

---

# 10. Load Balancer

Explain

Why needed

* Distribute traffic
* Health checks
* Failover

Algorithms

* Round Robin
* Least Connections
* Weighted

---

# 11. CDN

Useful for

* Images
* Videos
* CSS
* JS

Benefits

* Lower latency
* Less origin traffic

---

# 12. Message Queue

Examples

* Kafka
* RabbitMQ
* SQS

Used for

Notifications

Emails

Analytics

Logging

Video processing

Image compression

Background jobs

---

# 13. Storage

Discuss different storage.

### SQL

Users

Orders

Transactions

---

### NoSQL

Feeds

Posts

Logs

---

### Blob/Object Storage

Images

Videos

Files

---

# 14. Scaling

Horizontal scaling

```
1 Server

↓

10 Servers

↓

100 Servers
```

Vertical scaling

```
4 CPU

↓

64 CPU
```

Explain why horizontal scaling is generally preferred for web-scale systems.

---

# 15. Database Scaling

Read Replicas

```
Writes

↓

Primary DB

↓

Replica1

Replica2

Replica3
```

Reads go to replicas.

---

Sharding

```
Shard1

A-F

Shard2

G-L

Shard3

M-Z
```

or

Hash(UserID)

---

# 16. Bottlenecks

Identify possible problems.

Examples

Database hotspot

Cache misses

Large joins

Slow queries

Network latency

Queue backlog

Single point of failure

Large objects

---

# 17. Reliability

Replication

Retries

Circuit Breaker

Timeouts

Idempotency

Dead Letter Queue

Backup

Disaster Recovery

---

# 18. Security

Authentication

JWT

OAuth

Authorization

RBAC

Encryption

HTTPS

Encryption at rest

Rate limiting

CAPTCHA

Audit logs

---

# 19. Monitoring

Metrics

CPU

Memory

Latency

Error Rate

QPS

Dashboard

Prometheus

Grafana

Logging

ELK

Distributed tracing

OpenTelemetry

Alerts

PagerDuty

---

# 20. Trade-offs

This is where strong candidates stand out. There is rarely a single "correct" design—be explicit about why you chose one approach over another.

Examples:

| Choice        | Benefit                                    | Drawback                                                |
| ------------- | ------------------------------------------ | ------------------------------------------------------- |
| SQL           | Strong consistency, ACID transactions      | Harder to scale horizontally                            |
| NoSQL         | High scalability, flexible schema          | Limited joins, eventual consistency                     |
| Cache         | Lower latency, reduced DB load             | Cache invalidation complexity                           |
| CDN           | Faster content delivery                    | Added operational complexity                            |
| Message Queue | Decouples services, smooths traffic spikes | Increased eventual consistency and debugging complexity |
| Read Replicas | Higher read throughput                     | Replication lag                                         |

Discussing trade-offs demonstrates engineering judgment rather than just naming technologies.

---

# 21. Future Improvements

If time remains, mention potential enhancements:

* Multi-region deployment
* Disaster recovery
* Auto-scaling
* Search service (e.g., Elasticsearch/OpenSearch)
* Analytics pipeline
* Recommendation engine
* AI-powered features
* Cost optimization
* Feature flags and A/B testing

---

# A Complete Interview Flow (10-Step Template)

You can use this sequence for almost any system design interview:

1. **Clarify the problem** – Ask questions to define the scope and assumptions.
2. **List functional requirements** – What features must the system support?
3. **List non-functional requirements** – Scalability, latency, availability, consistency, security, etc.
4. **Estimate scale** – Users, requests per second, storage, bandwidth, and growth.
5. **Design the high-level architecture** – Clients, load balancers, services, databases, caches, queues, storage.
6. **Define APIs and data model** – Key endpoints and database schema.
7. **Explain request/data flow** – Walk through important operations step by step.
8. **Discuss scaling and optimization** – Caching, partitioning, replication, CDNs, asynchronous processing.
9. **Address reliability and security** – Failures, backups, monitoring, authentication, authorization.
10. **Summarize trade-offs and future improvements** – Explain why you made specific design decisions and what you'd improve as the system grows.

---

## Tips for Interviews

* Start with requirements instead of diving into architecture.
* State assumptions clearly when details are unspecified.
* Draw simple diagrams and explain them as you go.
* Think aloud so the interviewer can follow your reasoning.
* Prioritize the most critical components first; add complexity incrementally.
* Revisit requirements at the end to confirm your design satisfies them.
* Discuss alternatives and their trade-offs instead of presenting technologies as universally "best."

Following this structured flow helps ensure you cover the key areas interviewers expect while demonstrating a systematic approach to system design.

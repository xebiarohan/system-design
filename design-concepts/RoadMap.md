# System Design Roadmap

Think of the roadmap as **10 phases**:

```text
Phase 1  → System Design Fundamentals
Phase 2  → Networking & Communication
Phase 3  → API & Service Design
Phase 4  → Distributed Systems
Phase 5  → Messaging & Event-Driven Architecture
Phase 6  → Scalability & High Availability
Phase 7  → Reliability & Fault Tolerance
Phase 8  → Security
Phase 9  → Observability & Operations
Phase 10 → Architecture Patterns + Real System Design
```

---

# Phase 1 — System Design Fundamentals

Start here.

### 1. What is System Design?

* Functional requirements
* Non-functional requirements
* Constraints
* Scale
* Availability
* Latency
* Throughput
* Reliability
* Maintainability

### 2. Functional vs Non-Functional Requirements

Example:

```text
Functional:
    User can upload a photo

Non-functional:
    Upload should complete within 2 seconds
    System should support 10M users
    System should be available 99.99%
```

### 3. System Design Building Blocks

Understand the purpose of:

```text
Client
   ↓
Load Balancer
   ↓
Application Servers
   ↓
Cache
   ↓
Database
   ↓
Message Queue
   ↓
Object Storage
```

### 4. Vertical vs Horizontal Scaling

You already touched database scaling, but now understand scaling the **whole system**.

### 5. Stateless vs Stateful Services

Very important for distributed systems.

### 6. Monolith vs Modular Monolith vs Microservices

Understand:

* Monolith
* Modular monolith
* Microservices
* Service boundaries
* When microservices actually make sense

### 7. Synchronous vs Asynchronous Communication

```text
Sync:

Client → Service A → Service B → DB


Async:

Client → Service A → Queue → Service B
```

### 8. Latency vs Throughput

This becomes extremely important when evaluating architectures.

---

# Phase 2 — Networking & Communication

This is a **must-have** for system design.

You already know OAuth/TLS, so some of this will be familiar.

### 9. HTTP Fundamentals

* HTTP/1.1
* HTTP/2
* HTTP/3
* HTTP methods
* Headers
* Status codes
* Keep-alive
* Connection pooling

### 10. TCP vs UDP

Understand:

* Connection establishment
* Reliability
* Ordering
* Retransmission
* Flow control
* Congestion control

### 11. DNS

Understand:

```text
example.com
     ↓
DNS
     ↓
IP address
```

And:

* DNS records
* TTL
* DNS caching
* DNS load balancing
* Failover

### 12. TLS / HTTPS

You already studied this deeply through mTLS, so this should be a **review rather than a full topic**.

### 13. Reverse Proxy

Examples:

```text
Client
   ↓
Nginx / Envoy
   ↓
Application
```

Understand why reverse proxies exist.

### 14. Load Balancers

* L4 vs L7
* Round robin
* Least connections
* IP hashing
* Health checks
* Sticky sessions

### 15. CDN

Understand:

```text
User
 ↓
CDN
 ↓
Origin Server
```

Learn:

* Edge locations
* Cache hit/miss
* TTL
* Cache invalidation
* Static vs dynamic content

---

# Phase 3 — API & Service Design

This phase connects very nicely with your Spring Boot experience.

### 16. REST API Design

* Resource modeling
* HTTP methods
* Status codes
* Pagination
* Filtering
* Sorting
* Versioning

### 17. API Pagination

Understand:

```text
Offset pagination
Cursor pagination
Keyset pagination
```

And when each is appropriate.

### 18. API Versioning

Examples:

```text
/api/v1/users
/api/v2/users
```

Understand alternative strategies too.

### 19. Idempotency

**Extremely important.**

For example:

```http
POST /payments
Idempotency-Key: abc123
```

Why?

Because network failures can cause:

```text
Client → Payment Service
              ↓
           Payment succeeds
              ↓
        Response lost
              ↓
Client retries
```

Without idempotency, you could charge the customer twice.

### 20. Rate Limiting

Learn:

* Fixed window
* Sliding window
* Token bucket
* Leaky bucket

And distributed rate limiting using Redis.

### 21. API Gateway

Understand:

```text
                    ┌── User Service
Client → API Gateway ├── Order Service
                    ├── Payment Service
                    └── Product Service
```

Responsibilities:

* Routing
* Authentication
* Authorization
* Rate limiting
* Request transformation
* Logging

---

# Phase 4 — Distributed Systems

This is where system design gets really interesting.

### 22. What is a Distributed System?

Understand why distributed systems are fundamentally difficult.

### 23. Network Failures

Examples:

```text
Service A → Service B

Possible failures:

- timeout
- packet loss
- connection refused
- DNS failure
- partial response
- service unavailable
```

### 24. Timeouts

Learn why **every network call needs a timeout**.

### 25. Retries

Understand:

* Retry policies
* Exponential backoff
* Jitter
* Maximum retries

And the danger of retry storms.

### 26. Failure Detection

* Heartbeats
* Health checks
* Failure detection

### 27. Distributed Transactions

Learn:

* Two-phase commit
* 2PC problems
* Saga pattern
* Compensating transactions

### 28. Distributed Locks

Understand:

* Why locks are needed
* Redis-based locks
* Database locks
* Lock expiration
* Deadlocks

### 29. Leader Election

Conceptually understand:

```text
Node A
Node B
Node C

     ↓

Leader = Node B
```

### 30. Consensus

Learn the concepts behind:

* Raft
* Paxos

You don't initially need to implement them.

### 31. Distributed IDs

How systems generate unique IDs at huge scale.

Examples:

```text
UUID
Snowflake ID
Database sequence
```

Understand why auto-increment IDs become problematic in distributed systems.

---

# Phase 5 — Messaging & Event-Driven Architecture

This is another **major system-design topic**.

### 32. Message Queue vs Event Streaming

Understand the difference between:

```text
Queue
```

and

```text
Event Stream
```

### 33. Kafka Fundamentals

Since you're a Java/Spring developer, Kafka should be one of your practical technologies.

Learn:

* Producer
* Consumer
* Topic
* Partition
* Offset
* Consumer group
* Replication
* Ordering

### 34. Message Delivery Semantics

Very important:

```text
At-most-once
At-least-once
Exactly-once
```

### 35. Message Ordering

Understand why ordering becomes difficult with multiple consumers/partitions.

### 36. Dead Letter Queue

```text
Queue
 ↓
Consumer
 ↓
Failure
 ↓
Retry
 ↓
Retry
 ↓
DLQ
```

### 37. Event-Driven Architecture

Understand:

```text
Order Service
      ↓
   OrderCreated
      ↓
    Kafka
   ↙     ↘
Email    Inventory
Service   Service
```

### 38. Eventual Consistency

You already studied consistency models, so connect this concept to event-driven systems.

### 39. Outbox Pattern

**Very important system-design pattern.**

It solves the problem of:

```text
Database update succeeds
        +
Kafka publish fails
```

### 40. CDC — Change Data Capture

Understand how systems can capture database changes and publish them as events.

---

# Phase 6 — Scalability & High Availability

You've already studied database scaling, so now expand it to the entire architecture.

### 41. Horizontal Scaling

```text
          Load Balancer
          /     |     \
       App1   App2   App3
```

### 42. Stateless Architecture

Why stateless services scale much more easily.

### 43. Session Management

* Server-side sessions
* Distributed sessions
* Sticky sessions
* Token-based sessions

### 44. Caching Strategies

You already studied Redis, but now learn caching from the **system-design perspective**:

* Cache-aside
* Write-through
* Write-back
* Read-through
* Cache invalidation
* Cache stampede
* Cache penetration
* Cache avalanche

### 45. High Availability

Learn:

```text
Single instance
      ↓
Multiple instances
      ↓
Multiple AZs
      ↓
Multiple regions
```

### 46. Disaster Recovery

Learn:

* Backup
* Restore
* RPO
* RTO
* Failover
* Multi-region architecture

### 47. Multi-Region Systems

Understand:

```text
          Global Users
               ↓
        Global Load Balancer
          ↙           ↘
      Region A       Region B
```

---

# Phase 7 — Reliability & Fault Tolerance

This deserves its own phase.

### 48. Failure Isolation

### 49. Circuit Breaker

```text
Service A → Service B

        ↓ failures

Circuit OPEN

Service A ─X→ Service B
```

### 50. Bulkhead Pattern

Prevent one failing dependency from consuming all resources.

### 51. Retry + Backoff + Jitter

Go deeper than Phase 4.

### 52. Timeout Budgets

Understand why:

```text
Client timeout
    >
Service timeout
    >
DB timeout
```

must be designed carefully.

### 53. Load Shedding

When the system is overloaded, deliberately reject some work rather than allowing everything to fail.

### 54. Backpressure

Extremely important for high-throughput systems.

### 55. Graceful Degradation

For example:

```text
Recommendation service unavailable

Instead of:
❌ Entire website unavailable

Do:
✅ Show product
⚠️ Hide recommendations
```

### 56. SLO / SLA / SLI

Learn:

```text
SLI → measurement
SLO → target
SLA → contractual commitment
```

---

# Phase 8 — System Security

You've already studied OAuth/OIDC, TLS, mTLS, SAML, MFA and Passkeys, so you're actually ahead here.

Still cover system-design security:

### 57. Authentication vs Authorization

### 58. OAuth2 / OIDC

Review from an architecture perspective.

### 59. Service-to-Service Authentication

* mTLS
* OAuth2 client credentials
* Service identity

### 60. Secrets Management

Understand:

* Vault
* Cloud secret managers
* Key rotation
* Secret injection

### 61. Encryption

```text
At rest
In transit
```

### 62. Security Boundaries

* Network segmentation
* Zero Trust
* API gateway
* Service authorization

### 63. OWASP API Security

Focus particularly on:

* Broken authorization
* Authentication problems
* Injection
* Rate limiting
* Sensitive data exposure

---

# Phase 9 — Observability & Operations

This is often neglected by people preparing for system-design interviews.

Don't skip it.

### 64. Logging

* Structured logging
* Correlation IDs
* Centralized logging

### 65. Metrics

Examples:

```text
CPU
Memory
Request rate
Error rate
Latency
Queue depth
Cache hit ratio
```

### 66. Distributed Tracing

Understand:

```text
Request
 ↓
API Gateway
 ↓
Order Service
 ↓
Payment Service
 ↓
Database
```

and how a single trace follows the request across services.

### 67. OpenTelemetry

Learn the concepts of:

* Traces
* Spans
* Metrics
* Logs

### 68. Health Checks

* Liveness
* Readiness
* Startup probes

### 69. Monitoring & Alerting

Understand what should trigger alerts versus what should simply be observed.

---

# Phase 10 — Architecture Patterns + Real System Design

Now you start putting everything together.

### 70. Layered Architecture

```text
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

### 71. Hexagonal Architecture

### 72. Clean Architecture

### 73. Event-Driven Architecture

### 74. CQRS

```text
Write Model
     ↓
   Events
     ↓
Read Model
```

### 75. Event Sourcing

Understand how it differs from CQRS.

### 76. Saga Pattern

### 77. Strangler Fig Pattern

Useful for migrating monoliths.

### 78. API Composition

### 79. Backend-for-Frontend (BFF)

This connects nicely with your React + Spring Boot experience.

### 80. Service Mesh

Understand:

* Sidecars
* mTLS
* Traffic management
* Observability

You don't need to become a Kubernetes expert yet.

---

# Then: Real System Design Problems

**Only after the above foundation**, start designing complete systems.

I'd go roughly in this progression:

### Level 1 — Basic

1. URL Shortener
2. Pastebin
3. Rate Limiter
4. File Upload Service
5. Notification Service

### Level 2 — Intermediate

6. Chat System
7. News Feed
8. E-commerce backend
9. Ticket Booking System
10. Payment System
11. Job Scheduler
12. Distributed Cache

### Level 3 — Advanced

13. YouTube-like Video Platform
14. Netflix-like Streaming Platform
15. Uber-like Ride System
16. Instagram-like Social Network
17. WhatsApp-like Messaging
18. Distributed Search System
19. Food Delivery System
20. Large-scale Notification Platform

---

# The Complete Roadmap

So your overall system-design learning path becomes:

```text
DATABASE
────────────────────────────
1.  SQL vs NoSQL                 ✓
2.  Indexing                     ✓
3.  Transactions / ACID          ✓
4.  Isolation                    ✓
5.  Replication                  ✓
6.  Sharding                     ✓
7.  CAP                          ✓
8.  Consistency                  ✓
9.  Database Scaling             ✓
10. Caching / Redis              ✓
11. Read / Write Patterns        ✓
12. Polyglot Persistence         ✓


SYSTEM DESIGN
────────────────────────────
PHASE 1  System Design Fundamentals
PHASE 2  Networking
PHASE 3  API & Service Design
PHASE 4  Distributed Systems
PHASE 5  Messaging & Event Driven
PHASE 6  Scalability & HA
PHASE 7  Reliability
PHASE 8  Security
PHASE 9  Observability
PHASE 10 Architecture Patterns
PHASE 11 Real-world System Design
```

## One important recommendation for you

Don't treat this as **80 unrelated topics**.

Build the mental model:

```text
                    ┌──────────────┐
                    │   Clients    │
                    └──────┬───────┘
                           │
                        DNS/CDN
                           │
                     Load Balancer
                           │
                    API Gateway/BFF
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
          Service A     Service B     Service C
             │             │             │
             └──────┬──────┴──────┬──────┘
                    │             │
                  Cache         Kafka
                    │             │
                    ↓             ↓
                 Database      Consumers
                    │
             Replication/Sharding
                    │
                 Storage
```

Then layer **reliability, security, observability, and scaling** around that architecture.

That's the point where system design starts feeling like one coherent subject rather than a giant collection of buzzwords.

### What I'd suggest you do next

Since you've just completed the database roadmap, start with:

> **Phase 1 — System Design Fundamentals**

and go topic by topic exactly like we did with the database roadmap.

I'd make the first mini-roadmap:

1. What is System Design?
2. Functional vs Non-Functional Requirements
3. Scalability
4. Availability
5. Reliability
6. Latency vs Throughput
7. Stateless vs Stateful
8. Vertical vs Horizontal Scaling
9. Synchronous vs Asynchronous Communication
10. Monolith vs Microservices

Then we can take **Topic 1 — What is System Design?** and go deep with Java/Spring examples and a small architecture diagram.

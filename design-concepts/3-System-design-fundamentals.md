# Phase 1 — System Design Fundamentals

## 1. What is System Design?

System design is the process of deciding:

> **How should different software components work together to satisfy functional requirements at a particular scale while meeting non-functional requirements?**

Suppose someone says:

> "Build an e-commerce system."

That's not enough information to start coding.

You need to think about:

```text
Users
  ↓
Frontend
  ↓
API
  ↓
Services
  ↓
Cache
  ↓
Database
  ↓
Message Queue
  ↓
Other Services
```

And then ask:

* How many users?
* How many requests per second?
* How much data?
* How fast should responses be?
* What happens if a server crashes?
* What happens if the database goes down?
* Can we scale to 10× traffic?
* Do we need strong consistency?
* Can some operations be asynchronous?
* How do we secure the system?
* How do we monitor it?

That's **system design**.

---

## A simple example

Imagine you're building a URL shortener.

User sends:

```http
POST /urls

{
    "url": "https://example.com/some/very/long/url"
}
```

The system returns:

```json
{
    "shortUrl": "https://short.io/aB72x"
}
```

At small scale, this could simply be:

```text
Client
  ↓
Spring Boot
  ↓
PostgreSQL
```

But suppose you now have:

```text
100 million users
1 billion URLs
100,000 requests/second
```

Your architecture might become:

```text
                     ┌───────────┐
                     │   Client  │
                     └─────┬─────┘
                           ↓
                         CDN
                           ↓
                     Load Balancer
                           ↓
                 ┌─────────┼─────────┐
                 ↓         ↓         ↓
              Spring     Spring    Spring
              Boot       Boot      Boot
                 │         │         │
                 └────┬────┴────┬────┘
                      ↓         ↓
                    Redis    PostgreSQL
                               │
                          Replicas
```

The **business functionality hasn't changed**.

It's still:

> "Convert long URL → short URL."

But the system design has changed dramatically because of **scale and non-functional requirements**.

That's one of the most important ideas in system design.

---

# 2. Functional vs Non-Functional Requirements

This distinction is absolutely fundamental.

## Functional requirements

Functional requirements describe:

> **What the system should do.**

For an e-commerce application:

```text
User can register
User can login
User can search products
User can add products to cart
User can place order
User can make payment
User can track order
```

These are functionalities.

---

## Non-functional requirements

Non-functional requirements describe:

> **How the system should behave.**

For example:

```text
Response time < 200 ms
99.99% availability
Support 10 million users
Support 50,000 requests/sec
Data must not be lost
System should tolerate server failures
```

These are characteristics of the system.

---

## Example

Suppose you have:

```text
POST /orders
```

### Functional requirement

The API should create an order.

### Non-functional requirements

Maybe:

```text
Response time:
p95 < 300 ms

Availability:
99.99%

Traffic:
20,000 requests/sec

Durability:
Once payment succeeds, order must not disappear.
```

These requirements influence your architecture.

---

## Why this matters

Suppose your application currently looks like:

```text
Spring Boot
    ↓
PostgreSQL
```

That might satisfy:

```text
100 users
100 requests/sec
```

But perhaps not:

```text
10 million users
100,000 requests/sec
99.99% availability
```

You may need:

```text
Load Balancer
      ↓
Multiple application instances
      ↓
Redis
      ↓
Database replicas
      ↓
Message broker
```

So:

> **Requirements drive architecture.**

This is probably the most important system-design principle to remember.

---

# 3. Scalability

Scalability means:

> **The ability of a system to handle increasing workload by adding resources or changing the architecture.**

Suppose your application currently handles:

```text
1,000 requests/sec
```

Now traffic becomes:

```text
10,000 requests/sec
```

Can your architecture handle it?

If yes, the system scales.

---

## Two dimensions of scale

You can scale based on:

### Users

```text
1,000 users
      ↓
100,000 users
      ↓
10 million users
```

### Traffic

```text
100 req/sec
      ↓
10,000 req/sec
      ↓
1,000,000 req/sec
```

You also need to consider:

* Data volume
* Concurrent connections
* Storage
* Network traffic
* CPU
* Memory

---

## Example

Suppose:

```text
Spring Boot
    ↓
PostgreSQL
```

Your application handles:

```text
500 requests/sec
```

Traffic grows to:

```text
5,000 requests/sec
```

You might add multiple application instances:

```text
                 Load Balancer
                 /     |     \
                /      |      \
           App-1     App-2     App-3
              \        |        /
               \       |       /
                 PostgreSQL
```

Now application processing can scale horizontally.

But then the database might become the bottleneck.

So you might add:

```text
Redis
Read Replicas
Database Sharding
```

This is why system design is about **the entire system**, not just one component.

---

# 4. Availability

Availability means:

> **How much of the time is the system operational and accessible to users.**

Usually expressed as a percentage.

For example:

```text
99%
99.9%
99.99%
99.999%
```

These numbers look deceptively similar.

But the downtime difference is significant.

Approximate annual downtime:

| Availability | Downtime/year |
| ------------ | ------------: |
| 99%          |     3.65 days |
| 99.9%        |    8.76 hours |
| 99.99%       |  52.6 minutes |
| 99.999%      |  5.26 minutes |

So if someone says:

> "Our system needs 99.99% availability."

That's a significant architectural requirement.

---

## How do we improve availability?

### Single server

```text
User
 ↓
Server
 ↓
DB
```

If server crashes:

```text
❌ System unavailable
```

---

### Multiple servers

```text
              Load Balancer
              /           \
             ↓             ↓
          Server 1       Server 2
             \             /
              \           /
                 Database
```

If Server 1 crashes:

```text
              Load Balancer
                    ↓
                 Server 2
```

The system continues operating.

---

## Availability through redundancy

This is a major system-design concept:

> **Avoid a Single Point of Failure (SPOF).**

If you have:

```text
A → B → C
```

and B is the only instance of B:

```text
A → B → C
    ↑
   SPOF
```

If B fails, everything fails.

Instead:

```text
       ┌── B1 ──┐
A ─────┤        ├──── C
       └── B2 ──┘
```

Now there is redundancy.

---

# 5. Reliability

Availability and reliability are related but **not identical**.

### Availability

> Is the system available?

### Reliability

> Does the system consistently perform correctly over time, including under failures?

Imagine an API:

```text
POST /payment
```

It is available:

```text
HTTP 200
```

But sometimes the customer gets charged twice.

The service is **available**, but the system is not behaving reliably.

---

## Reliability includes things like:

* Correctness
* Fault tolerance
* Data durability
* Consistent behavior
* Recovery from failures
* Handling unexpected conditions

---

## Example: Payment

Imagine:

```text
Client
   ↓
Payment Service
   ↓
Bank
```

The bank processes the payment successfully.

But the network connection breaks before your service receives the response.

Your application doesn't know whether payment succeeded.

If the client retries:

```text
Client
 ↓
Payment Service
 ↓
Bank
 ↓
Payment succeeds

Network failure

Client retries
 ↓
Bank
 ↓
Payment succeeds AGAIN
```

Now you've charged twice.

A reliable design might use:

```text
Idempotency Key
```

For example:

```http
POST /payments
Idempotency-Key: abc123
```

The payment service remembers:

```text
abc123 → payment result
```

If the same request arrives again:

```text
abc123
```

it returns the existing result instead of performing another payment.

This is the kind of thinking system design requires.

---

# 6. Latency vs Throughput

This is one of the most important concepts.

## Latency

Latency means:

> **How long does one operation take?**

Example:

```text
GET /products
```

takes:

```text
120 ms
```

That's latency.

---

## Throughput

Throughput means:

> **How much work can the system process in a given amount of time?**

Usually:

```text
requests/sec
transactions/sec
messages/sec
```

Example:

```text
System processes 10,000 requests/sec
```

That's throughput.

---

## Simple analogy

Imagine a coffee shop.

### Latency

How long does **one customer** wait for their coffee?

```text
2 minutes
```

### Throughput

How many coffees can the shop produce?

```text
100 coffees/hour
```

They're related but different.

---

## High latency, high throughput

A system could have:

```text
Latency = 2 seconds
Throughput = 100,000 requests/sec
```

For example, a large batch-processing system.

Conversely:

```text
Latency = 10 ms
Throughput = 100 requests/sec
```

could be a small low-latency service.

---

## Percentiles

In real system design, don't only say:

> "Average latency is 100 ms."

Because averages can hide bad experiences.

You often see:

```text
p50 = 50 ms
p95 = 120 ms
p99 = 300 ms
p99.9 = 1 sec
```

Meaning:

### p50

50% of requests are faster than:

```text
50 ms
```

### p95

95% are faster than:

```text
120 ms
```

### p99

99% are faster than:

```text
300 ms
```

The remaining 1% take longer.

For large-scale systems, **tail latency** matters enormously.

---

# 7. Stateless vs Stateful

This is a very important architectural concept.

## Stateful service

A stateful server remembers information about the client.

Example:

```text
Client
  ↓
Server 1

Server 1 remembers:
sessionId = ABC
user = Rohan
cart = [Laptop]
```

If the next request goes to Server 2:

```text
Client
  ↓
Server 2
```

Server 2 doesn't know about that state.

Problem.

---

## Stateless service

A stateless server doesn't rely on local memory to remember the user's session.

For example:

```text
Client
  ↓
Server 1

Client
  ↓
Server 2

Client
  ↓
Server 3
```

Any server can handle the request.

State might be stored externally:

```text
Client
   ↓
Any application server
   ↓
Redis / Database
```

---

## Why stateless architecture is useful

Suppose:

```text
             Load Balancer
             /     |     \
            ↓      ↓      ↓
          App1   App2   App3
```

Because the applications are stateless, you can easily add:

```text
App4
App5
App6
```

when traffic increases.

---

## Spring Boot example

Imagine you store:

```java
private Map<String, UserSession> sessions;
```

inside your application.

That's local state.

If:

```text
Request 1 → App1
Request 2 → App2
```

App2 won't have App1's memory.

Instead:

```text
App1 ──┐
App2 ──┼── Redis
App3 ──┘
```

Now all instances share the state.

---

## Important distinction

Stateless doesn't mean:

> "The system has no state."

It means:

> **The individual application instance doesn't own the state required to process future requests.**

The state can live in:

* Database
* Redis
* Object storage
* External session store
* Another service

This distinction is worth remembering.

---

# 8. Vertical vs Horizontal Scaling

You already encountered scaling in your database roadmap, but now let's apply it to application architecture.

---

## Vertical scaling

Increase the power of a single machine.

For example:

```text
Before:

4 CPU
8 GB RAM


After:

32 CPU
128 GB RAM
```

Architecture remains:

```text
          Server
             │
             ↓
          Database
```

You're making the machine bigger.

---

## Horizontal scaling

Add more machines.

Instead of:

```text
          Server
```

you have:

```text
       Load Balancer
        /    |    \
       ↓     ↓     ↓
     App1   App2   App3
```

Each server might have:

```text
4 CPU
8 GB RAM
```

If traffic increases:

```text
App4
App5
App6
```

can be added.

---

## Vertical scaling

Advantages:

* Simple
* Often requires fewer architectural changes
* Easy to understand

Disadvantages:

* Hardware has limits
* Expensive at the high end
* Single point of failure
* Scaling often requires a restart/downtime

---

## Horizontal scaling

Advantages:

* Can scale much further
* Better fault tolerance
* Easier to distribute traffic
* Supports elastic scaling

Disadvantages:

* More complex
* Requires load balancing
* Requires statelessness or distributed state
* Distributed-system problems appear

---

# Putting the 8 concepts together

Now let's connect everything.

Suppose you're designing:

> **An online shopping system for 10 million users.**

You start with requirements.

### Functional

```text
User registration
Product search
Cart
Order
Payment
Order tracking
```

### Non-functional

```text
High availability
Low latency
Large scale
Reliable payments
```

Then your architecture starts evolving.

```text
                        Users
                          │
                          ↓
                         CDN
                          │
                          ↓
                    Load Balancer
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
          App-1        App-2        App-3
             │            │            │
             └────────────┼────────────┘
                          ↓
                        Redis
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
          Product       Order       Payment
          Service       Service      Service
             │            │            │
             ↓            ↓            ↓
          Product DB    Order DB    Payment DB
                          │
                          ↓
                        Kafka
                          │
              ┌───────────┼───────────┐
              ↓           ↓           ↓
           Email      Inventory    Analytics
```

Now notice what we learned:

### Functional requirements

Tell us **what** the system does.

### Non-functional requirements

Tell us **how well** it must do it.

### Scalability

Allows us to handle increasing traffic.

### Availability

Ensures the system remains accessible despite failures.

### Reliability

Ensures the system behaves correctly and recovers from failures.

### Latency

Tells us how quickly individual operations complete.

### Throughput

Tells us how much work the system can process.

### Statelessness

Allows application instances to be easily distributed.

### Horizontal scaling

Allows us to add more instances as traffic increases.

---

# The mental model I want you to build

When you get a system-design question, **don't immediately start drawing Kafka, Redis, Kubernetes, microservices, etc.**

Start with:

```text
1. What does the system need to do?
                  ↓
2. How much traffic/data?
                  ↓
3. What latency is required?
                  ↓
4. What availability is required?
                  ↓
5. What reliability guarantees are needed?
                  ↓
6. Where are the bottlenecks?
                  ↓
7. How should we scale?
                  ↓
8. What happens when components fail?
```

Then choose technologies.

**Requirements → Constraints → Architecture → Components → Scaling → Failure handling**

That sequence is much more important than memorizing architecture diagrams.

And since you've already completed **SQL/NoSQL, replication, sharding, CAP, consistency, Redis, read/write patterns, and polyglot persistence**, you're now in a very good position to start connecting those database concepts to these system-level concepts.

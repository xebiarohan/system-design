
# What is Backpressure?

Backpressure is a mechanism where a system tells its upstream component:

> **"I'm overloaded. Slow down or stop sending me more work."**

It's a way for a slower component to **protect itself** from being overwhelmed.

---

# A Real-Life Analogy

Imagine a restaurant.

```text
Customers
    ↓
Waiter
    ↓
Kitchen
```

The kitchen can cook:

```text
20 dishes/minute
```

But customers suddenly order:

```text
100 dishes/minute
```

The kitchen cannot keep up.

Without backpressure:

```text
Orders pile up

↓

Kitchen gets overwhelmed

↓

Long delays

↓

Mistakes

↓

Restaurant fails
```

With backpressure:

```text
Kitchen

↓

"Stop taking new orders for a few minutes."
```

The waiter slows down the incoming orders.

That is backpressure.

---

# Why Do We Need It?

Imagine your application:

```text
Users

↓

API

↓

Message Queue

↓

Worker

↓

Database
```

The worker processes:

```text
100 jobs/sec
```

Suddenly users create:

```text
1000 jobs/sec
```

Now:

```text
Queue grows

↓

Memory usage increases

↓

Latency increases

↓

Eventually system crashes
```

Backpressure prevents this.

---

# Another Example

Suppose you're building an image-processing service.

User uploads:

```text
Image
```

The upload service places jobs into a queue.

```text
Upload Service

↓

Queue

↓

Image Processor
```

Image processor speed:

```text
10 images/sec
```

Uploads:

```text
500 images/sec
```

Without backpressure:

```text
Queue

10

100

1,000

10,000

100,000

...
```

Memory usage explodes.

Eventually:

```text
Out of Memory

↓

Crash
```

---

# With Backpressure

The processor says:

```text
"I'm overloaded."
```

Now the upload service:

* stops accepting uploads temporarily,
* slows down producers, or
* rejects new uploads.

The queue stays manageable.

---

# Difference Between Rate Limiting and Backpressure

This is a favorite interview question.

| Rate Limiting                                       | Backpressure                                   |
| --------------------------------------------------- | ---------------------------------------------- |
| Protects the system from abuse or excessive usage   | Protects downstream components from overload   |
| Based on predefined limits                          | Based on current system capacity               |
| Usually applied at the edge (gateway/load balancer) | Usually applied between internal components    |
| Example: 100 requests/min/user                      | Example: Queue is full, stop sending more jobs |

---

## Example

Suppose your API allows:

```text
100 requests/minute
```

A user sends:

```text
1000 requests
```

Rate limiter:

```text
Reject after 100.
```

---

Now imagine:

Only:

```text
50 requests
```

But the database is already overloaded.

The rate limiter says:

```text
50 < 100

Allowed
```

But the database says:

```text
"I'm overloaded!"
```

Backpressure activates.

---

# Where Does Backpressure Happen?

Usually **inside distributed systems**.

Example:

```text
API

↓

Queue

↓

Worker

↓

Database
```

Possible backpressure points:

```text
Queue → API

Database → Worker

Worker → Queue
```

Any slower component can apply backpressure.

---

# Common Backpressure Strategies

There isn't just one way to implement backpressure.

## 1. Reject Requests

Simplest method.

When overloaded:

```text
Queue Full

↓

Reject new request
```

Response:

```http
503 Service Unavailable
```

or

```http
429 Too Many Requests
```

depending on the situation.

---

## 2. Slow Down Producers

Instead of rejecting:

```text
Producer

↓

Sleep 100 ms

↓

Next request
```

The producer naturally generates fewer requests.

This is common in stream-processing systems.

---

## 3. Block the Producer

The producer waits.

Example:

```text
Queue Full

↓

Producer waits

↓

Worker finishes job

↓

Producer continues
```

No data is lost.

Trade-off:

Higher latency.

---

## 4. Buffer Requests

Temporary storage.

```text
Producer

↓

Buffer

↓

Consumer
```

Good when overload lasts only a few seconds.

Problem:

Buffers can eventually become full.

---

## 5. Drop Requests

Common in streaming.

Example:

```text
Live Video Frames
```

If processing can't keep up:

```text
Frame 101

Frame 102

Frame 103
```

Drop:

```text
Frame 101
```

Continue with newer frames.

Missing one frame is better than freezing the stream.

---

# Queue-Based Example

Imagine:

```text
Producer

↓

Kafka Topic

↓

Consumer
```

Producer speed:

```text
1000 messages/sec
```

Consumer speed:

```text
100 messages/sec
```

Without backpressure:

```text
Kafka

100

1000

10,000

100,000 messages
```

Eventually storage fills up.

---

With backpressure:

Consumer informs the producer:

```text
Slow down.
```

Producer reduces rate.

---

# Database Example

Suppose:

```text
Worker

↓

Database
```

Database can handle:

```text
500 writes/sec
```

Workers send:

```text
2000 writes/sec
```

Without backpressure:

```text
Database

↓

Slow queries

↓

Connection pool exhausted

↓

Timeouts
```

With backpressure:

Workers reduce write speed.

Database remains healthy.

---

# Real-World Examples

### Video Streaming

Video player:

```text
Internet

↓

Video Buffer

↓

Player
```

If the internet becomes slow:

The player pauses downloading requests until the buffer has room.

This is a form of backpressure.

---

### Apache Kafka

Consumers process messages at their own pace.

If consumers are slow:

* the producer can be slowed,
* or messages accumulate in the topic until retention limits are reached.

---

### Reactive Streams

Frameworks like **Project Reactor** or **RxJava** have built-in backpressure.

Consumer:

```text
Give me only:

100 items
```

Producer sends exactly:

```text
100
```

After processing:

```text
Request next 100
```

Instead of flooding the consumer.

---

# System Design Perspective

Suppose you're designing a notification service.

Architecture:

```text
User Action

↓

API

↓

Queue

↓

Notification Workers

↓

Email Service
```

Suddenly:

```text
1 million notifications
```

The email provider can send:

```text
10,000 emails/minute
```

Without backpressure:

```text
Queue grows forever

↓

Workers overload

↓

Memory increases

↓

Failures
```

With backpressure:

```text
Queue becomes too large

↓

Workers signal overload

↓

API slows down or temporarily rejects notification requests

↓

System remains stable
```

---

# Interview Mental Model

When you hear **Backpressure**, think:

```text
A downstream component is slower than an upstream component.

Producer
     │
     ▼
Consumer can't keep up
     │
     ▼
Consumer says:
"Slow down!"
```

---

# Rate Limiting vs. Backpressure

This distinction is worth memorizing because interviewers ask it often.

| Scenario                                      | Rate Limiting | Backpressure |
| --------------------------------------------- | ------------- | ------------ |
| A user sends 10,000 API requests              | ✅ Yes         | ❌ No         |
| Database is overloaded                        | ❌ No          | ✅ Yes        |
| Prevent API abuse                             | ✅ Yes         | ❌ No         |
| Queue is full                                 | ❌ No          | ✅ Yes        |
| Protect login endpoint from password guessing | ✅ Yes         | ❌ No         |
| Consumer is slower than producer              | ❌ No          | ✅ Yes        |

### The simplest way to remember it

* **Rate Limiting** asks: **"Is the client allowed to send this many requests?"**
* **Backpressure** asks: **"Can the next component in the system handle any more work right now?"**

One enforces **policy**; the other responds to **current system capacity**. That's why modern distributed systems often use **both** together.

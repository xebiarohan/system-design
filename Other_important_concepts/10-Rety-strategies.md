# Retry strategies

Retry strategies are a fundamental technique in distributed systems because **failures are normal, not exceptional**. Networks are unreliable, services restart, databases get temporarily overloaded, and packets can be lost.

A retry strategy answers the question:

> **If an operation fails, when and how should we try again?**

---

## Why do we need retries?

Imagine an order service calling a payment service.

```text
Order Service  ------------->  Payment Service
                     Pay ₹100
```

Normally:

```text
Order Service  -----> Payment Service
                     Success
```

But many things can go wrong.

### Temporary failures

* Network timeout
* Temporary database lock
* Service restarting
* High CPU
* Short-lived overload

These are often solved by retrying.

Example:

```text
Attempt 1
↓

Timeout

↓

Attempt 2

↓

Success
```

Without retries:

```text
Customer sees Payment Failed
```

even though the service would have recovered one second later.

---

# But retries can also be dangerous

Suppose a service is already overloaded.

```
10,000 requests/sec
```

It starts timing out.

Clients think:

> "I'll retry."

Now they send

```
20,000 requests/sec
```

The service becomes even slower.

Clients retry again.

```
40,000 requests/sec
```

Now you've created a **retry storm**.

Instead of helping, retries made the outage worse.

This is why retry strategies are an important part of system design.

---

# Which errors should be retried?

Not every failure should trigger a retry.

### Retry

Temporary failures

```
Timeout
Connection reset
503 Service Unavailable
429 Too Many Requests
Temporary network issue
```

These often succeed on a later attempt.

---

### Don't Retry

Permanent failures

```
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
Validation Error
```

Retrying these will never help because the request itself is invalid.

---

# Types of Retry Strategies

## 1. Immediate Retry

Simply retry immediately.

```
Attempt 1

↓

Failed

↓

Attempt 2 immediately

↓

Success
```

Example

```java
for(int i=0;i<3;i++){

    if(call())
        break;
}
```

### Advantages

* Very simple
* Good for tiny transient glitches

### Problems

If the service is overloaded,

```
Retry immediately
Retry immediately
Retry immediately
```

adds even more pressure.

Usually not recommended.

---

# 2. Fixed Delay Retry

Wait a constant amount of time.

Example

```
Attempt 1

↓

Wait 2 sec

↓

Attempt 2

↓

Wait 2 sec

↓

Attempt 3
```

This gives the service some time to recover.

---

Example

```java
retry

sleep(2 seconds)

retry

sleep(2 seconds)

retry
```

Better than immediate retry but still not ideal.

---

# 3. Exponential Backoff (Most Popular)

Instead of waiting a fixed amount,

increase the wait after every failure.

Example

```
Attempt 1

↓

Wait 1 sec

↓

Attempt 2

↓

Wait 2 sec

↓

Attempt 3

↓

Wait 4 sec

↓

Attempt 4

↓

Wait 8 sec
```

Notice the pattern

```
1

2

4

8

16
```

The delay doubles each time.

### Why?

If a service is overloaded,

don't hammer it continuously.

Give it progressively more time to recover.

This is the retry strategy you'll hear about most often in interviews.

---

# 4. Exponential Backoff with Jitter (Industry Standard)

Imagine 100,000 clients fail at the same time.

Without jitter

```
Everyone retries after exactly 1 second.
```

```
100000 requests

↓

1 second later

↓

100000 requests again
```

The server gets overloaded again.

Now everyone waits

```
2 seconds
```

Again,

```
100000 requests together
```

This creates synchronized spikes.

---

Instead, add randomness.

Instead of

```
2 seconds
```

clients wait

```
1.8 sec

2.3 sec

1.5 sec

2.1 sec
```

Now retries are spread out.

```
Time

|

Requests

|

||||||||||||

↓

|

||||

↓

|

||

↓

|

|
```

Much smoother.

This is called **jitter**.

Large companies like Amazon, Google, and Netflix recommend exponential backoff with jitter because it prevents retry storms.

---

# Maximum Retry Count

Never retry forever.

Example

```
Maximum retries = 5
```

```
Attempt 1

↓

Attempt 2

↓

Attempt 3

↓

Attempt 4

↓

Attempt 5

↓

Give up
```

Otherwise,

a bug or outage could cause infinite retries.

---

# Retry Timeout

Also place a limit on total retry time.

Example

```
Retry for at most 30 seconds
```

After that,

fail the request.

---

# Retry + Circuit Breaker

Imagine the payment service is completely down.

Without a circuit breaker

```
Every request

↓

Retry 5 times

↓

Retry 5 times

↓

Retry 5 times
```

The service never gets a chance to recover.

Instead

```
Failures increase

↓

Circuit opens

↓

Stop sending requests

↓

Wait 30 seconds

↓

Try again
```

Retries and circuit breakers are commonly used together.

---

# Retry + Idempotency

This is one of the most important interview topics.

Suppose

```
Pay ₹100
```

The server processes the payment successfully.

But the response is lost because of a network timeout.

The client thinks

```
Payment failed
```

So it retries.

Now the customer is charged twice.

To avoid this,

clients send an **idempotency key**, a unique identifier for the operation.

```
Request

Idempotency-Key: abc123
```

The server stores the result associated with that key.

If the same request arrives again with the same key,

```
Idempotency-Key: abc123
```

the server returns the previous result instead of processing the payment again.

Retries become safe because repeating the same request doesn't repeat the side effect.

---

# Retry Flow in a Real System

Consider an e-commerce application.

```text
User
   |
   v
Order Service
   |
   v
Payment Service
```

1. Order Service sends a payment request.
2. Payment Service times out.
3. Order Service waits using exponential backoff with jitter.
4. It retries up to a maximum number of attempts.
5. Each retry includes the same idempotency key.
6. If the payment service had already completed the payment, it returns the stored result instead of charging again.
7. If all retries fail, the request is marked as failed or queued for later processing.

---

# Common Interview Trade-offs

| Strategy                     | Advantages                                                        | Disadvantages                           |
| ---------------------------- | ----------------------------------------------------------------- | --------------------------------------- |
| Immediate Retry              | Simple, fast for tiny glitches                                    | Can overload struggling services        |
| Fixed Delay                  | Easy to implement                                                 | All clients retry at the same intervals |
| Exponential Backoff          | Reduces pressure over time                                        | Can increase overall latency            |
| Exponential Backoff + Jitter | Avoids synchronized retry spikes and is widely used in production | Slightly more complex to implement      |

## Mental model for system design interviews

When discussing retries, think through these questions in order:

1. **Is the error temporary or permanent?** Retry only temporary failures.
2. **How long should I wait?** Prefer exponential backoff with jitter.
3. **How many times should I retry?** Set a maximum retry count or overall timeout.
4. **Is retrying safe?** Use idempotency for operations with side effects (payments, order creation, etc.).
5. **What if the service is completely down?** Combine retries with a circuit breaker to avoid making the outage worse.

This combination—**exponential backoff + jitter + retry limits + idempotency + circuit breaker**—is the pattern you'll find in many production systems and is a strong answer in system design interviews.

Idempotency is one of the **most important concepts** in distributed systems because it makes **retries safe**.

The formal definition is:

> **An operation is idempotent if performing it multiple times has the same effect as performing it once.**

That sounds abstract, so let's understand it with examples.

---

# Example 1: Turning on a light

Suppose you have a light switch.

```text
OFF

↓

Turn ON

↓

ON
```

If you press "Turn ON" again:

```text
ON

↓

Turn ON

↓

Still ON
```

Nothing changes.

So **"Turn ON" is idempotent.**

---

# Example 2: Adding money to an account

Imagine your account balance is:

```text
Balance = ₹1000
```

You send:

```text
Add ₹100
```

Result:

```text
₹1100
```

Now imagine the client doesn't receive the response (due to a timeout) and retries.

```text
Add ₹100
```

Balance becomes:

```text
₹1200
```

Oops!

The money was added twice.

This operation is **not idempotent**.

---

# Why is this a problem?

Consider an online payment.

```text
Customer

↓

Pay ₹500

↓

Payment Service
```

The payment service successfully charges the card.

```text
Success
```

But before the response reaches the client:

```text
Network timeout
```

The client thinks:

> "The payment failed."

So it retries.

```text
Pay ₹500
```

The payment service charges the card **again**.

The customer has now paid **₹1000** instead of ₹500.

---

# The solution: Idempotency Keys

Instead of identifying a request only by its contents, the client sends a unique identifier.

Example:

```http
POST /payments

Idempotency-Key: 8f3d-92ab-xyz
```

The first time the server receives this request:

```text
Key = 8f3d-92ab-xyz

↓

Process payment

↓

Store result
```

The server saves something like:

| Idempotency Key | Result             |
| --------------- | ------------------ |
| 8f3d-92ab-xyz   | Payment Successful |

---

## What happens on a retry?

The client retries with the **same** key.

```http
POST /payments

Idempotency-Key: 8f3d-92ab-xyz
```

Instead of processing the payment again, the server:

1. Looks up the key.
2. Finds it already exists.
3. Returns the **previous response**.

```text
Already processed

↓

Return Success
```

No second charge happens.

---

# Visual Flow

Without idempotency:

```text
Client

↓

Pay ₹500

↓

Server

↓

Charged

↓

Timeout

↓

Client retries

↓

Server charges again ❌
```

With idempotency:

```text
Client

↓

Pay ₹500
Key = abc123

↓

Server

↓

Charged

↓

Timeout

↓

Client retries
Key = abc123

↓

Server

↓

Key already exists

↓

Return previous response ✅
```

---

# Where are idempotency keys stored?

Usually in a fast database or cache like **Redis**.

Example:

```text
Key: abc123

↓

Payment ID: P1001

↓

Status: SUCCESS

↓

Response: {...}
```

When another request comes with the same key:

```text
Lookup

↓

Found

↓

Return stored response
```

---

# Which operations need idempotency?

Generally, **operations that create side effects**.

Examples:

* Payment processing
* Order creation
* Ticket booking
* Money transfer
* Sending invoices

Imagine creating an order:

```text
Create Order
```

If retried without idempotency:

```text
Order #101

Order #102
```

The customer accidentally places two identical orders.

With idempotency:

```text
Create Order

↓

Order #101

↓

Retry

↓

Return Order #101
```

---

# HTTP Methods and Idempotency

Some HTTP methods are naturally idempotent.

| HTTP Method | Idempotent?  | Why?                                                           |
| ----------- | ------------ | -------------------------------------------------------------- |
| GET         | ✅ Yes        | Reading data doesn't change state.                             |
| PUT         | ✅ Usually    | Replaces a resource with the same data each time.              |
| DELETE      | ✅ Usually    | Deleting an already deleted resource has no additional effect. |
| POST        | ❌ Usually No | Often creates a new resource each time it's called.            |

### Example

`GET /users/1`

You can call it 100 times.

Still the same user.

---

`DELETE /users/1`

First request:

```text
User deleted
```

Second request:

```text
User already doesn't exist
```

Still deleted.

No extra side effect.

---

`POST /orders`

Every call creates:

```text
Order 101

Order 102

Order 103
```

Not idempotent by default.

---

# Does POST have to be non-idempotent?

No.

If you use an idempotency key:

```http
POST /orders

Idempotency-Key: xyz123
```

Then retries won't create duplicate orders.

Many payment APIs (such as those from major payment providers) use this pattern.

---

# How long should the server store idempotency keys?

Not forever.

Typically, they're stored with an expiration time (TTL).

Example:

```text
Key

↓

Stored for 24 hours

↓

Automatically removed
```

The TTL depends on the business use case.

---

# What if the client changes the request but reuses the same key?

Suppose the first request is:

```text
Idempotency-Key: abc123

Amount: ₹500
```

Later, the client sends:

```text
Idempotency-Key: abc123

Amount: ₹1000
```

This is a bug or misuse.

The server should **reject** it (often with an error like HTTP 409 Conflict or a validation error), because the same idempotency key must always represent the **same logical operation**.

---

# Real-world example: Flight booking

Imagine booking a flight.

```text
Book Seat

↓

Seat Reserved

↓

Network Timeout
```

The app retries.

Without idempotency:

```text
Booking #101

Booking #102
```

You might end up with two reservations.

With idempotency:

```text
Booking Request

↓

Booking #101

↓

Retry

↓

Return Booking #101
```

Only one booking exists.

---

# Interview Mental Model

Whenever you hear:

* **Retries**
* **Payments**
* **Orders**
* **Money transfer**
* **Booking systems**
* **Distributed systems**

Immediately ask yourself:

> **"What happens if the client retries after the server has already completed the operation?"**

If the answer is **"duplicate side effects could occur,"** then the operation should be made idempotent.

### A common production pattern

```text
Client
   |
   | POST /payments
   | Idempotency-Key: abc123
   v
API Server
   |
   | Check key in Redis/Database
   |
   +--> Key exists?
   |      |
   |      +--> Yes → Return stored response
   |      |
   |      +--> No
   |             |
   |             +--> Process payment
   |             +--> Store key + response
   |             +--> Return success
```

This pattern ensures that **multiple identical requests produce a single business action**, even if the client retries due to timeouts or temporary failures.

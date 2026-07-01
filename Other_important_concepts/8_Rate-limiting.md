# What is Rate Limiting?

Rate limiting is a mechanism that **controls how many requests a user/client can make within a specific time period**.

Examples:

* Max **100 requests per minute** per user
* Max **10 login attempts per hour**
* Max **1000 API calls per day**
* Max **5 password reset requests per minute**

Without rate limiting, a single client could overwhelm your system.

---

# Why Do We Need Rate Limiting?

Imagine you're running an API.

Without limits:

```
User A -> 10 requests/sec
User B -> 15 requests/sec
Attacker -> 100,000 requests/sec
```

The attacker consumes all resources:

* CPU
* Memory
* Database connections
* Network bandwidth

Result:

```
Normal users experience:
❌ Slow responses
❌ Timeouts
❌ Service outages
```

Rate limiting protects your system.

---

# Real-World Examples

### Login Endpoint

```
POST /login
```

Limit:

```
5 attempts per minute
```

Why?

Prevents brute-force attacks.

---

### Public API

Example:

```
GET /users
```

Limit:

```
100 requests/minute
```

Why?

Prevents abuse.

---

### OTP Service

Limit:

```
3 OTPs per hour
```

Why?

SMS costs money.

---

# Where Can Rate Limiting Be Applied?

## Per User

```
User 123
→ 100 requests/min
```

---

## Per IP Address

```
192.168.1.1
→ 50 requests/min
```

---

## Per API Key

```
API Key ABC
→ 1000 requests/day
```

---

## Per Organization

```
Company X
→ 1 million requests/day
```

---

# What Happens When Limit Is Exceeded?

The request is rejected.

HTTP response:

```http
429 Too Many Requests
```

Example:

```json
{
  "error": "Rate limit exceeded"
}
```

Sometimes servers also return:

```http
Retry-After: 60
```

Meaning:

```
Wait 60 seconds and try again.
```

---

# Core Challenge

Suppose:

```
Limit = 100 requests/minute
```

How do we know whether a user has already made 100 requests?

We need an algorithm.

This is where interview questions start.

---

# Algorithm 1: Fixed Window Counter

Most intuitive approach.

Rule:

```
100 requests per minute
```

Store:

```text
User A:
Current Minute = 10:01
Count = 87
```

When request arrives:

```python
count += 1

if count > 100:
    reject
```

At:

```
10:02
```

Reset counter.

---

## Example

Minute:

```
10:01
```

Requests:

```
1
2
3
...
100
```

101st request:

```
Reject
```

---

## Problem

Boundary issue.

User sends:

```
100 requests at 10:01:59
100 requests at 10:02:00
```

Total:

```
200 requests in 1 second
```

But limiter allows it.

This is called the **fixed window burst problem**.

---

# Algorithm 2: Sliding Window Log

More accurate.

Store timestamp of every request.

Example:

```
10:01:01
10:01:05
10:01:20
10:01:50
```

When a new request arrives:

Remove timestamps older than 60 seconds.

Count remaining timestamps.

If count >= limit:

```
Reject
```

Else:

```
Allow
```

---

## Example

Limit:

```
3 requests/minute
```

Stored:

```
10:00:10
10:00:30
10:00:50
```

New request:

```
10:01:00
```

Window:

```
Last 60 seconds only
```

Now:

```
10:00:10 removed
```

Remaining:

```
10:00:30
10:00:50
```

Request allowed.

---

## Advantage

Much more accurate.

---

## Disadvantage

Need to store every request timestamp.

For millions of users:

```
Huge memory cost
```

---

# Algorithm 3: Sliding Window Counter

Hybrid solution.

Instead of storing every request:

Store counters for windows.

Example:

```
Current Window = 80 requests
Previous Window = 60 requests
```

Calculate weighted count.

Provides near-sliding-window accuracy.

Uses less memory.

Many production systems use variations of this.

---

# Algorithm 4: Token Bucket (Very Important)

This is probably the most popular interview algorithm.

Think of a bucket.

```
Capacity = 100 tokens
```

Every request consumes:

```
1 token
```

Tokens refill continuously.

Example:

```
10 tokens/sec
```

---

## Initial State

```
Bucket = 100 tokens
```

User sends:

```
20 requests
```

Remaining:

```
80 tokens
```

---

## Heavy Traffic

User sends:

```
120 requests
```

First:

```
100 allowed
```

Then:

```
20 rejected
```

---

## Refill

After:

```
1 second
```

Add:

```
10 tokens
```

Bucket:

```
10 tokens available
```

---

## Why Companies Love It

Allows small bursts while still protecting the system.

Traffic isn't perfectly smooth in real life.

Users naturally burst.

Example:

```
Open app
→ 20 API requests immediately
```

Token bucket handles this well.

---

# Algorithm 5: Leaky Bucket

Imagine a bucket leaking water at a fixed speed.

Requests enter:

```
Fast
```

Requests leave:

```
Constant rate
```

Example:

```
Incoming:
100 requests instantly
```

Processed:

```
10/sec
10/sec
10/sec
```

This smooths traffic.

Useful when backend systems need steady load.

---

# Where Is Rate Limiting Implemented?

## API Gateway

Most common.

```
Client
   ↓
API Gateway
   ↓
Services
```

Gateway checks limits before requests hit services.

Examples:

* NGINX
* Kong
* Envoy

---

## Load Balancer Layer

```
Client
 ↓
Load Balancer
 ↓
Servers
```

Can block excessive traffic early.

---

## Application Layer

Inside service code:

```python
if requests > limit:
    return 429
```

Simple but consumes application resources before rejecting.

---

# Distributed System Challenge

Single server is easy.

But what if you have:

```
10 API servers
```

User sends:

```
50 requests → Server A
50 requests → Server B
50 requests → Server C
```

Each server thinks:

```
Only 50 requests
```

But actual total:

```
150 requests
```

Limit broken.

---

# Solution: Shared Store

Use a centralized fast datastore.

Usually:

* Redis

Architecture:

```text
User
  ↓
API Server
  ↓
Redis Counter
```

Redis stores:

```
user123 = 87 requests
```

All servers check the same counter.

Now limits remain accurate.

---

# Interview-Level Design

If asked:

> Design a rate limiter for 100M users

A strong answer usually includes:

1. API Gateway layer
2. Token Bucket algorithm
3. Redis for shared counters
4. 429 responses
5. Per-user and per-IP limits
6. Horizontal scaling of Redis
7. Monitoring and analytics

Architecture:

```text
Client
   ↓
Load Balancer
   ↓
API Gateway
   ↓
Rate Limiter
   ↓
Redis
   ↓
Services
```

# Mental Model

Whenever you hear **Rate Limiting**, think:

```text
Protect resources
      ↓
Track request counts
      ↓
Apply an algorithm
      ↓
Reject excess traffic (429)
      ↓
Use Redis/shared storage in distributed systems
```

This topic connects directly to the next concepts in your roadmap:

**Rate Limiting → Backpressure → Retry Strategies → Idempotency**

Understanding how a system handles "too much traffic" is the foundation for all four.




This is actually one of the most important parts of rate limiting because in system design interviews you'll often be asked:

> "Where would you place the rate limiter?"

The answer depends on **what you're trying to protect** and **how early you want to reject bad traffic**.

---

# Big Picture

Imagine a typical web system:

```text
Client
   ↓
Load Balancer
   ↓
API Gateway
   ↓
Application Services
   ↓
Database
```

A request travels through multiple layers.

Rate limiting can be placed at different points.

---

# 1. Rate Limiting at the Load Balancer

Architecture:

```text
Client
   ↓
Load Balancer
   ↓
Application Servers
```

Example:

```text
1. Client sends request
2. Load balancer receives it
3. Load balancer checks limit
4. Allow or reject
```

---

## Why do this?

Because it's the earliest point inside your infrastructure.

If you reject here:

```text
CPU saved
Memory saved
Network saved
Application resources saved
```

The request never reaches your services.

---

## Example

Suppose someone launches a DDoS attack:

```text
500,000 requests/sec
```

Without LB rate limiting:

```text
Load Balancer
    ↓
Application Servers
    ↓
Database
```

Everything gets flooded.

---

With LB rate limiting:

```text
Load Balancer
    ↓
Reject 95% immediately
```

Most traffic never enters your system.

---

## What is commonly limited?

### Per IP

```text
IP A → 100 req/sec
IP B → 100 req/sec
```

Very common.

---

## Advantages

✔ Stops traffic early

✔ Saves infrastructure resources

✔ Good for attack protection

---

## Disadvantages

Load balancers usually don't know:

```text
User ID
API key
Subscription tier
```

They only see network-level information.

So they can't easily enforce:

```text
Premium User = 1000 req/min
Free User = 100 req/min
```

---

# 2. Rate Limiting at API Gateway (Most Common)

Architecture:

```text
Client
   ↓
API Gateway
   ↓
Microservices
```

Examples of API gateways:

* Kong
* Envoy
* Apache APISIX

---

Think of API Gateway as:

```text
Security Guard
Traffic Controller
Rate Limiter
Authentication Layer
```

all in one place.

---

## Request Flow

```text
Client
   ↓
API Gateway
   ↓
Auth Check
   ↓
Rate Limit Check
   ↓
Service
```

---

Example:

```text
User 123
Limit = 100 requests/min
```

Gateway stores:

```text
user123 -> 57 requests
```

New request arrives:

```text
count = 58
```

Allowed.

---

At:

```text
count = 101
```

Gateway returns:

```http
429 Too Many Requests
```

before contacting any backend service.

---

## Why Companies Prefer This

Imagine:

```text
User Service
Order Service
Payment Service
Notification Service
```

Without gateway:

```text
Each service
implements its own limiter
```

Messy.

---

With gateway:

```text
Single rate limiter
for all services
```

Centralized.

---

## Advantages

✔ Centralized

✔ Easy to manage

✔ Knows user identity

✔ Knows API key

✔ Knows subscription plan

✔ Protects backend services

---

## Disadvantages

Gateway can become:

```text
Single Point of Failure
```

unless replicated.

---

Production setup:

```text
           Gateway 1
Client →   Gateway 2
           Gateway 3

               ↓
            Redis
```

All gateways share counters.

---

# 3. Rate Limiting Inside Application Services

Architecture:

```text
Client
   ↓
Gateway
   ↓
Application Service
```

The service itself checks limits.

Example:

```python
def create_post(user_id):

    requests = redis.get(user_id)

    if requests > 100:
        return 429

    process_request()
```

---

## Why do this?

Sometimes limits are business-specific.

Example:

```text
Free users:
5 AI generations/day

Paid users:
1000 AI generations/day
```

Only the application understands these rules.

---

Example:

```text
POST /generate-image
```

The service knows:

```text
User Plan
Credits Remaining
Monthly Quota
```

The gateway may not.

---

## Advantages

Very flexible.

Can enforce business logic.

---

## Disadvantages

The request already reached:

```text
Application Server
```

Resources have already been consumed.

Not ideal for heavy abuse.

---

# 4. Database-Level Rate Limiting

Less common.

Sometimes databases themselves protect against excessive usage.

Example:

```text
Maximum connections = 1000
```

After that:

```text
New connections rejected
```

---

But this is usually considered:

```text
Last line of defense
```

not true rate limiting.

If traffic reaches the DB, you've already spent lots of resources.

---

# Real Production Architecture

A large company often uses multiple layers.

Example:

```text
Internet
   ↓
CDN
   ↓
Load Balancer
   ↓
API Gateway
   ↓
Services
   ↓
Database
```

Rate limits may exist at:

```text
CDN
↓
Load Balancer
↓
API Gateway
↓
Application Service
```

simultaneously.

---

# Example: Designing Instagram's Rate Limiting

Suppose a user opens the app.

Requests:

```text
GET /feed
GET /stories
GET /notifications
GET /messages
```

Instagram might use:

### Layer 1: CDN

```text
Protect against bots
```

---

### Layer 2: Load Balancer

```text
1000 req/sec/IP
```

Protect infrastructure.

---

### Layer 3: API Gateway

```text
100 req/min/user
```

Protect backend services.

---

### Layer 4: Application

```text
Create Post:
10 posts/hour

Send Message:
500 messages/day
```

Business rules.

---

# Interview Perspective

If an interviewer asks:

> "Where would you implement rate limiting?"

A strong answer is:

```text
Infrastructure protection
    → Load Balancer / API Gateway

User/API quotas
    → API Gateway

Business-specific limits
    → Application Layer
```

For most modern distributed systems, the **API Gateway + Redis** approach is the standard answer because it centralizes rate limiting, scales horizontally, and prevents unnecessary load on backend services.

Once you're comfortable with this, the next topic (**Backpressure**) becomes much easier because rate limiting and backpressure are both mechanisms for handling overload—but they operate at different stages of the request flow.

Prevents attack like



DDoS attack → "I want to make your service unavailable."

Brute force attack → "I want to break into someone's account."



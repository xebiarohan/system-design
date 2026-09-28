# Phase 2 — Networking & Communication

## Topic 1 — HTTP Fundamentals

HTTP is the protocol that allows **clients and servers to communicate over a network**.

At the simplest level:

```text
Client                         Server
  |                              |
  | -------- HTTP Request ------>|
  |                              |
  | <------- HTTP Response ------|
  |                              |
```

For example, when your React application calls your Spring Boot backend:

```text
React
  |
  | GET /api/users/123
  |
  v
Spring Boot
  |
  | HTTP Response
  |
  v
React
```

But there is much more going on underneath this.

---

# 1. What exactly is HTTP?

**HTTP = HyperText Transfer Protocol**

It is an **application-layer protocol** used for communication between applications.

For example:

```text
React Browser
     |
     | HTTP
     v
Spring Boot API
```

HTTP defines things such as:

* How a request is structured
* How a response is structured
* Methods such as GET, POST, PUT, DELETE
* Status codes such as 200, 404, 500
* Headers
* Cookies
* Caching
* Content negotiation
* Connection behavior

HTTP itself doesn't care whether the server is written in:

```text
Java
Node.js
Python
Go
C#
```

As long as the server understands HTTP.

---

# 2. HTTP sits on top of TCP/IP

This is **very important for System Design**.

When you make:

```text
GET https://example.com/users
```

there are multiple networking layers involved.

A simplified view:

```text
┌──────────────────────────┐
│ Application              │
│ HTTP                     │
├──────────────────────────┤
│ Transport                │
│ TCP                      │
├──────────────────────────┤
│ Internet                 │
│ IP                       │
├──────────────────────────┤
│ Network Access           │
│ Ethernet / Wi-Fi         │
└──────────────────────────┘
```

HTTP is therefore **not responsible for everything**.

For example:

**HTTP**

```text
GET /users
Host: example.com
```

**TCP**

Handles reliable delivery of bytes.

**IP**

Handles addressing/routing between machines.

This distinction becomes extremely useful later when we discuss:

* Load balancers
* Reverse proxies
* TCP vs UDP
* TLS
* HTTP/2
* HTTP/3
* Connection pooling
* Latency

---

# 3. HTTP Request

Let's look at an actual HTTP request.

```http
GET /api/users/123 HTTP/1.1
Host: example.com
Authorization: Bearer eyJ...
Accept: application/json
User-Agent: Mozilla/5.0
```

An HTTP request consists primarily of:

```text
Request
│
├── Method
├── URL / Path
├── HTTP Version
├── Headers
└── Body
```

Let's break it down.

---

# 4. HTTP Method

The method tells the server **what operation the client wants to perform**.

Common methods:

| Method  | Typical purpose                   |
| ------- | --------------------------------- |
| GET     | Retrieve data                     |
| POST    | Create/process something          |
| PUT     | Replace/update resource           |
| PATCH   | Partially update resource         |
| DELETE  | Delete resource                   |
| HEAD    | Retrieve headers only             |
| OPTIONS | Ask what operations are supported |

Example:

```http
GET /users/123
```

means:

> Give me user 123.

While:

```http
DELETE /users/123
```

means:

> Delete user 123.

---

# 5. GET

GET is normally used to **retrieve data**.

```http
GET /api/products/100
```

Response:

```json
{
  "id": 100,
  "name": "Laptop",
  "price": 1200
}
```

A key property of GET is that it should be **safe**.

Meaning:

```text
GET /products/100
```

shouldn't cause something like:

```text
Delete product
Transfer money
Create order
```

GET requests are also commonly cacheable.

For example:

```text
Browser
   |
   | GET /products
   v
Cache
   |
   | cache hit
   v
Response
```

This becomes important when we study HTTP caching.

---

# 6. POST

POST is generally used when the client wants the server to **create something or perform an operation**.

Example:

```http
POST /api/orders
Content-Type: application/json
```

Body:

```json
{
  "productId": 100,
  "quantity": 2
}
```

Server:

```text
Create Order
     |
     v
Order ID = 9876
```

Response:

```http
HTTP/1.1 201 Created
```

```json
{
  "orderId": 9876
}
```

POST is generally **not idempotent**.

If you send:

```text
POST /orders
```

twice, you might create:

```text
Order #101
Order #102
```

This becomes very important in distributed systems when network failures cause retries.

We'll come back to this.

---

# 7. PUT

PUT is generally used to **replace the representation of a resource**.

Example:

```http
PUT /users/123
```

```json
{
  "name": "Rohan",
  "email": "rohan@example.com"
}
```

Conceptually:

```text
Existing user
       ↓
Replace with
       ↓
New representation
```

PUT is normally **idempotent**.

If you execute:

```text
PUT /users/123
```

with the same request five times, the resulting resource should be the same.

```text
PUT
PUT
PUT
PUT
PUT
 ↓
same final state
```

This distinction is **very important for system design**.

---

# 8. PATCH

PATCH is generally used for a **partial modification**.

Suppose the user is:

```json
{
  "name": "Rohan",
  "email": "rohan@example.com",
  "age": 35
}
```

You only want to change the age.

```http
PATCH /users/123
```

```json
{
  "age": 36
}
```

You aren't sending the complete user representation.

---

# 9. DELETE

Used to remove a resource.

```http
DELETE /users/123
```

Response might be:

```http
HTTP/1.1 204 No Content
```

Again, DELETE is normally considered **idempotent**.

For example:

```text
DELETE /users/123
```

First call:

```text
User deleted
```

Second call:

```text
User already doesn't exist
```

The final state is still:

```text
User does not exist
```

---

# 10. HTTP Headers

Headers carry **metadata** about the request or response.

Example:

```http
GET /api/users/123 HTTP/1.1
Host: example.com
Accept: application/json
Authorization: Bearer xyz
User-Agent: Mozilla/5.0
```

Some important headers:

### Content-Type

Tells the server what the request body contains.

```http
Content-Type: application/json
```

Meaning:

```text
Body = JSON
```

---

### Accept

Tells the server what response format the client wants.

```http
Accept: application/json
```

---

### Authorization

Used to send credentials/tokens.

```http
Authorization: Bearer eyJhbGciOi...
```

You've already encountered this in your OAuth2 learning.

The flow becomes:

```text
React
 |
 | Authorization: Bearer <access-token>
 v
Spring Boot
 |
 | validate token
 v
Resource
```

---

### Cache-Control

Controls caching.

```http
Cache-Control: max-age=3600
```

Meaning the response can be cached for 3600 seconds.

---

### Cookie

Used to send cookies.

```http
Cookie: sessionId=abc123
```

---

# 11. HTTP Request Body

Some requests contain a body.

For example:

```http
POST /users HTTP/1.1
Content-Type: application/json

{
  "name": "Rohan",
  "age": 35
}
```

The body is:

```json
{
  "name": "Rohan",
  "age": 35
}
```

Typically:

```text
GET       → usually no body
POST      → commonly has body
PUT       → commonly has body
PATCH     → commonly has body
DELETE    → usually no body
```

These are conventions, not absolute protocol restrictions.

---

# 12. HTTP Response

The server responds with something like:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "Rohan"
}
```

A response contains:

```text
Response
│
├── Status Code
├── Headers
└── Body
```

---

# 13. HTTP Status Codes

This is another thing you absolutely need for system design.

Status codes are grouped into five categories.

```text
1xx → Informational
2xx → Success
3xx → Redirection
4xx → Client error
5xx → Server error
```

---

## 2xx — Success

### 200 OK

Successful request.

```http
HTTP/1.1 200 OK
```

Common for:

```text
GET
PUT
PATCH
```

---

### 201 Created

Resource successfully created.

```http
POST /users

→ 201 Created
```

---

### 204 No Content

Request succeeded but there is no response body.

Common example:

```http
DELETE /users/123

→ 204 No Content
```

---

# 14. 4xx — Client Errors

These generally indicate that something about the client's request is wrong.

### 400 Bad Request

Malformed/invalid request.

```http
POST /users
```

```json
{
  "age": "hello"
}
```

Could result in:

```http
400 Bad Request
```

---

### 401 Unauthorized

This one is frequently misunderstood.

It generally means:

> Authentication is missing or invalid.

Example:

```text
No access token
       ↓
401
```

---

### 403 Forbidden

Authentication may be valid, but the caller isn't allowed to perform the operation.

```text
Valid token
    +
Insufficient permission
    ↓
403
```

This connects directly to your OAuth2 work.

For example:

```text
JWT
 ↓
sub = user123
roles = USER
 ↓
POST /admin/users
 ↓
403 Forbidden
```

---

### 404 Not Found

Resource doesn't exist.

```http
GET /users/999999
```

```http
404 Not Found
```

---

### 409 Conflict

The request conflicts with the current state.

Example:

```text
Create username = rohan
```

but:

```text
username already exists
```

Could return:

```http
409 Conflict
```

---

### 429 Too Many Requests

Rate limit exceeded.

```text
Client
  |
  | 1000 requests/sec
  v
API
  |
  | Rate limit = 100 req/sec
  v
429
```

This is extremely important in system design.

---

# 15. 5xx — Server Errors

These generally indicate a server-side failure.

### 500 Internal Server Error

Generic server failure.

```text
Request
   ↓
Spring Boot
   ↓
Unexpected exception
   ↓
500
```

---

### 502 Bad Gateway

Very important when discussing proxies/load balancers.

Imagine:

```text
Client
  |
  v
Load Balancer
  |
  v
Backend
```

If the load balancer acts as a gateway and gets an invalid response from the upstream server, it may return:

```text
502 Bad Gateway
```

---

### 503 Service Unavailable

Server/service currently cannot handle the request.

Possible reasons:

```text
Service overloaded
Service down
Maintenance
No available backend
```

---

### 504 Gateway Timeout

Gateway/proxy didn't receive a timely response from the upstream service.

```text
Client
  |
  v
Load Balancer
  |
  v
Service
  |
  |------ too slow ------X
  |
Load Balancer timeout
  |
  v
504
```

This becomes **very important later when we study distributed systems and microservices**.

---

# 16. HTTP is Stateless

This is one of the **most important system-design concepts**.

HTTP itself is stateless.

Suppose:

```text
Request 1:
GET /profile

Request 2:
GET /orders

Request 3:
GET /cart
```

The server doesn't inherently remember:

```text
Request 1 came from Rohan
Request 2 came from the same Rohan
Request 3 came from the same Rohan
```

Each HTTP request contains whatever information is needed to process it.

For example:

```http
Authorization: Bearer <JWT>
```

Then:

```text
Request
   ↓
JWT
   ↓
Server identifies user
```

This is one reason **stateless services scale horizontally so well**.

---

# 17. Stateless HTTP Service

Imagine:

```text
                Load Balancer
                     |
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Server A   Server B   Server C
```

Request 1:

```text
Client → Server A
```

Request 2:

```text
Client → Server C
```

Request 3:

```text
Client → Server B
```

If the application is stateless, this is perfectly fine.

Every server can process the request.

This is exactly the concept you learned earlier in **System Design Phase 1 — Stateless vs Stateful Services**.

---

# 18. HTTP and Cookies

HTTP itself is stateless, but applications can build **stateful sessions** using cookies.

Example:

```text
Client
  |
  | POST /login
  v
Server
  |
  | Set-Cookie: SESSION_ID=abc123
  v
Client
```

Later:

```http
GET /profile
Cookie: SESSION_ID=abc123
```

Server:

```text
SESSION_ID
     ↓
Session Store
     ↓
User = Rohan
```

So:

```text
HTTP → stateless
Application → can maintain state using cookies/sessions
```

---

# 19. HTTP Keep-Alive

Imagine making 10 requests:

```text
GET /users
GET /orders
GET /products
GET /cart
...
```

Without connection reuse, you might repeatedly establish TCP connections.

Conceptually:

```text
TCP connection
   ↓
HTTP request
   ↓
HTTP response
   ↓
connection closed

TCP connection
   ↓
HTTP request
   ↓
...
```

That's expensive.

HTTP persistent connections allow the connection to be reused:

```text
TCP connection
   |
   ├── Request 1
   ├── Response 1
   ├── Request 2
   ├── Response 2
   ├── Request 3
   └── Response 3
```

This reduces connection overhead and latency.

---

# 20. HTTP/1.1

HTTP/1.1 is still very important to understand.

A simplified example:

```text
TCP Connection
      |
      ├── Request 1
      ├── Response 1
      ├── Request 2
      ├── Response 2
      └── Request 3
```

HTTP/1.1 supports persistent connections.

But there is an important limitation around request/response ordering that can lead to **head-of-line blocking**.

This motivated HTTP/2.

---

# 21. HTTP/2

HTTP/2 introduced **multiplexing**.

Instead of:

```text
Request 1 → Response 1
Request 2 → Response 2
Request 3 → Response 3
```

HTTP/2 can have multiple streams over one connection:

```text
              TCP Connection
                    |
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     Stream 1    Stream 2    Stream 3
       GET         GET         GET
```

Responses can be interleaved.

This dramatically improves efficiency for applications making many requests.

HTTP/2 also introduced concepts such as:

* Binary framing
* Multiplexing
* Header compression
* Stream prioritization

---

# 22. HTTP/3

HTTP/3 goes one step further.

Instead of:

```text
HTTP
 ↓
TCP
 ↓
IP
```

it uses:

```text
HTTP/3
   ↓
QUIC
   ↓
UDP
   ↓
IP
```

So:

```text
HTTP/1.1 → TCP
HTTP/2   → TCP
HTTP/3   → QUIC/UDP
```

QUIC provides reliability and other transport functionality while avoiding some limitations of TCP.

You'll want to understand this later, but **don't go too deep into HTTP/3 yet**.

---

# 23. HTTP vs HTTPS

HTTP itself doesn't encrypt the communication.

```text
HTTP
 ↓
Plain communication
```

HTTPS is essentially:

```text
HTTP
 +
TLS
```

Conceptually:

```text
Application
    ↓
HTTP
    ↓
TLS
    ↓
TCP
    ↓
IP
```

With HTTPS:

```text
Client
  |
  | encrypted
  v
Server
```

This protects against someone on the network reading or modifying the traffic.

This connects nicely with your previous OAuth/mTLS learning.

---

# 24. HTTP + TLS + Authentication are different things

This distinction is worth remembering:

```text
HTTPS
  ↓
Protects communication

OAuth2
  ↓
Authorization framework

OIDC
  ↓
Authentication layer

JWT
  ↓
Token format

mTLS
  ↓
Mutual certificate authentication
```

They solve different problems.

For example:

```text
React
   |
   | HTTPS
   | Authorization: Bearer <JWT>
   ↓
Spring Boot
```

Here:

```text
HTTPS → encrypts transport
JWT   → carries identity/authorization information
OAuth → defines how token is obtained/used
```

---

# 25. Idempotency — VERY Important for System Design

This is one of the HTTP concepts I especially want you to remember.

An operation is **idempotent** if repeating the same request produces the same intended final state.

For example:

```text
PUT /users/123

{
    "name": "Rohan"
}
```

Send it once:

```text
name = Rohan
```

Send it 10 times:

```text
name = Rohan
```

Final state remains:

```text
name = Rohan
```

That's idempotent.

But:

```text
POST /orders
```

might create:

```text
Order 101
Order 102
Order 103
```

if repeated.

---

# 26. Why Idempotency Matters in Distributed Systems

This is where HTTP becomes **system design**, rather than just API development.

Imagine:

```text
Client
   |
   | POST /payments
   v
Payment Service
   |
   | Payment successful
   |
   X
 Network failure
```

The client doesn't receive the response.

So the client thinks:

```text
"Did payment succeed?"
```

It retries:

```text
POST /payments
```

Now you could potentially charge the customer twice.

A common solution is an **idempotency key**:

```http
POST /payments
Idempotency-Key: abc-123
```

Server:

```text
abc-123 → Payment already processed
```

Therefore:

```text
Retry
  ↓
same idempotency key
  ↓
same operation/result
```

This is a **very important real-world system design concept**.

---

# 27. HTTP Caching

HTTP has built-in caching mechanisms.

Example:

```http
GET /products/100
```

Response:

```http
Cache-Control: max-age=3600
```

The client/browser/cache can reuse the response.

Architecture:

```text
Client
  |
  v
Cache
  |
  | cache miss
  v
Load Balancer
  |
  v
Application
  |
  v
Database
```

On subsequent requests:

```text
Client
  |
  v
Cache HIT
  |
  v
Response
```

This reduces:

* Latency
* Server load
* Database load
* Network traffic

Later in the roadmap we'll go much deeper into caching.

---

# 28. HTTP Content Negotiation

The client can tell the server what it can accept.

```http
Accept: application/json
```

Server:

```http
Content-Type: application/json
```

Another example:

```http
Accept-Encoding: gzip, br
```

Server might respond with:

```http
Content-Encoding: gzip
```

So the response is compressed.

This matters because large responses consume:

```text
Bandwidth
+
Network time
```

---

# 29. HTTP Request Lifecycle

Let's put everything together.

Suppose your React application does:

```javascript
fetch("/api/orders/123");
```

Conceptually:

```text
React
  |
  | HTTP GET
  v
DNS
  |
  | find server IP
  v
TCP / QUIC
  |
  | establish transport
  v
TLS
  |
  | HTTPS encryption
  v
Load Balancer
  |
  v
Spring Boot Server
  |
  ├── Authentication
  ├── Authorization
  ├── Controller
  ├── Service
  ├── Database
  |
  v
HTTP Response
  |
  v
Load Balancer
  |
  v
Client
```

This single picture is worth remembering.

---

# 30. HTTP in a Typical System Design

Suppose we're designing an e-commerce system:

```text
                 Internet
                    |
                    v
                 HTTPS
                    |
                    v
             Load Balancer
                    |
          ┌─────────┼─────────┐
          ↓         ↓         ↓
       API-1     API-2      API-3
          |         |         |
          └─────────┼─────────┘
                    |
                 Redis
                    |
                    v
                Database
```

HTTP is the communication protocol between many of these components:

```text
Browser → Load Balancer
Load Balancer → API
API → another service
```

For example:

```http
POST /orders
Authorization: Bearer xxx
Content-Type: application/json
Idempotency-Key: 12345
```

Response:

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
    "orderId": "ORD-123"
}
```

Now you can start seeing how the HTTP concepts connect to the larger system.

---

# 31. What you should remember for System Design

I'd put these into your notes:

```text
HTTP Fundamentals
│
├── HTTP = application-layer protocol
│
├── HTTP request
│   ├── Method
│   ├── URL/path
│   ├── Headers
│   └── Body
│
├── HTTP response
│   ├── Status code
│   ├── Headers
│   └── Body
│
├── Methods
│   ├── GET
│   ├── POST
│   ├── PUT
│   ├── PATCH
│   └── DELETE
│
├── Status codes
│   ├── 2xx → success
│   ├── 3xx → redirect
│   ├── 4xx → client error
│   └── 5xx → server error
│
├── Statelessness
│
├── Cookies / Sessions
│
├── Keep-Alive
│
├── Idempotency
│
├── Caching
│
├── HTTP/1.1
├── HTTP/2
└── HTTP/3
```

## ⭐ The 6 concepts I'd prioritize

For your **11-year Java/Spring + system design** level, don't spend equal time on every HTTP detail. Make sure these six are crystal clear:

**1. Request/Response structure**

```text
Method + URL + Headers + Body
              ↓
Status + Headers + Body
```

**2. Status codes**

Especially:

```text
200
201
204
400
401
403
404
409
429
500
502
503
504
```

**3. Statelessness**

Why stateless APIs make horizontal scaling easier.

**4. Idempotency**

Especially:

```text
PUT vs POST
```

and:

```text
Idempotency-Key
```

for retries/payment/order APIs.

**5. HTTP/1.1 vs HTTP/2 vs HTTP/3**

Understand the architectural differences:

```text
HTTP/1.1 → persistent connections
HTTP/2   → multiplexing over TCP
HTTP/3   → QUIC over UDP
```

**6. HTTP + TLS + Authentication**

Understand that these are different layers/responsibilities:

```text
HTTP
 ↓
communication

TLS
 ↓
secure transport

OAuth2/OIDC/JWT
 ↓
authentication/authorization
```

### One final mental model

If you remember only one diagram from this topic, remember this:

```text
                HTTP
                 │
       ┌─────────┴─────────┐
       │                   │
    REQUEST             RESPONSE
       │                   │
   ┌───┴────┐         ┌────┴────┐
   │        │         │         │
 Method  Headers    Status    Headers
   │        │         │         │
   └───┬────┘         └────┬────┘
       │                   │
      Body                Body
       │                   │
       └─────────┬─────────┘
                 │
              Network
                 │
          TCP / TLS / IP
```

That foundation will make the **next networking topics—DNS, TCP vs UDP, TLS, HTTP/2, WebSockets, and load balancing—much easier**, because you'll understand *what HTTP is doing and what the layers underneath it are responsible for*.

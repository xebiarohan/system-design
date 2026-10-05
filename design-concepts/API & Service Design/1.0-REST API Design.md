# REST API Design

## 1. What are we actually designing?

Suppose we have an e-commerce system.

We have:

* Users
* Products
* Orders
* Payments

A frontend might need APIs like:

```text
GET    /users/123
GET    /products/456
POST   /orders
GET    /orders/789
DELETE /orders/789
```

REST API design is about deciding:

> **How should clients interact with the resources exposed by our system?**

A good REST API should have:

* Clear resource modeling
* Predictable URLs
* Correct HTTP methods
* Meaningful HTTP status codes
* Consistent request/response formats
* Pagination for large collections
* Filtering and sorting
* A versioning strategy
* Well-defined error responses

---

# 2. Resource Modeling — the most important part

This is where many people initially make mistakes.

Think in terms of **resources**, not actions.

Suppose you want to create a user.

### ❌ RPC-style

```http
POST /createUser
```

or:

```http
POST /users/create
```

The URL is describing an **action**.

Instead:

### ✅ REST-style

```http
POST /users
```

Because `users` is the resource.

The HTTP method tells us what we're doing with that resource.

```text
POST /users
     ↑
   create
```

Similarly:

```http
GET /users
```

means:

> Give me users.

And:

```http
GET /users/123
```

means:

> Give me user 123.

---

# 3. Think of URLs as nouns

A useful mental model:

```text
URL = Resource
HTTP method = Operation
```

For example:

| Operation             | API                 |
| --------------------- | ------------------- |
| Get all users         | `GET /users`        |
| Get one user          | `GET /users/123`    |
| Create user           | `POST /users`       |
| Replace user          | `PUT /users/123`    |
| Partially update user | `PATCH /users/123`  |
| Delete user           | `DELETE /users/123` |

Notice that the URL stays focused on the resource.

---

# 4. Resource hierarchy

Suppose an order belongs to a user.

You could have:

```http
GET /users/123/orders
```

Meaning:

> Get orders belonging to user 123.

And:

```http
GET /users/123/orders/456
```

Meaning:

> Get order 456 belonging to user 123.

This expresses the relationship:

```text
User
 └── Orders
      └── Order
```

But be careful not to create ridiculously deep URLs.

### ❌

```text
/users/123/orders/456/items/789/products/555/reviews/10
```

That's becoming painful.

Usually, once you have the resource ID, a flatter API is easier:

```http
GET /orders/456
GET /products/555
GET /reviews/10
```

---

# 5. HTTP Methods

Your roadmap explicitly includes HTTP methods as part of REST design. 

The important ones are:

```text
GET
POST
PUT
PATCH
DELETE
```

Let's understand them properly.

---

## GET

Used to retrieve data.

```http
GET /users/123
```

Response:

```json
{
  "id": 123,
  "name": "Rohan",
  "email": "rohan@example.com"
}
```

GET should not modify server state.

So this would be bad:

```http
GET /users/123/delete
```

Use:

```http
DELETE /users/123
```

instead.

---

# 6. POST

POST is generally used when the server is asked to create a new resource or perform an operation that doesn't fit an idempotent update.

Example:

```http
POST /users
```

Request:

```json
{
  "name": "Rohan",
  "email": "rohan@example.com"
}
```

Response:

```http
201 Created
```

Possibly:

```http
Location: /users/123
```

The server generated the ID:

```text
123
```

---

# 7. PUT vs PATCH

This is a **very common interview question**.

### PUT

Generally represents replacing the resource representation at a known URI.

```http
PUT /users/123
```

Request:

```json
{
  "name": "Rohan",
  "email": "new@example.com",
  "phone": "123456789"
}
```

Conceptually:

```text
Old User
   ↓
Replace
   ↓
New User
```

### PATCH

Used for partial modification.

```http
PATCH /users/123
```

Request:

```json
{
  "email": "new@example.com"
}
```

Only the email changes.

```text
name     → unchanged
email    → changed
phone    → unchanged
```

### Interview shortcut

Think:

```text
PUT   → replace
PATCH → partial modification
```

There are nuances in the HTTP specifications around what a particular PUT/PATCH API semantics means, but this is the useful design distinction.

---

# 8. DELETE

Used to remove a resource.

```http
DELETE /users/123
```

Possible response:

```http
204 No Content
```

No response body is necessary.

---

# 9. HTTP Status Codes

This is another major part of REST API design.

Don't return:

```http
200 OK
```

for absolutely everything.

Use status codes to communicate what happened.

## 2xx — Success

### 200 OK

Successful request.

```http
GET /users/123

200 OK
```

### 201 Created

A resource was created.

```http
POST /users

201 Created
```

### 204 No Content

Successful operation with no response body.

Common for:

```http
DELETE /users/123
```

---

# 10. 4xx — Client-side problems

### 400 Bad Request

The request itself is invalid.

Example:

```json
{
  "age": "abc"
}
```

when `age` must be numeric.

---

### 401 Unauthorized

This one is confusing.

It essentially means:

> The request lacks valid authentication credentials.

For example:

```text
No access token
Invalid access token
Expired access token
```

---

### 403 Forbidden

Authentication exists, but the caller isn't allowed to perform the operation.

Example:

```text
User authenticated
        ↓
        ✓
Needs ADMIN role
        ↓
        ✗
403 Forbidden
```

This fits nicely with your OAuth/OIDC knowledge.

---

### 404 Not Found

Resource doesn't exist.

```http
GET /users/999999
```

If the user doesn't exist:

```http
404 Not Found
```

---

### 409 Conflict

The request conflicts with the current state.

Example:

```http
POST /users
```

with:

```json
{
  "email": "existing@example.com"
}
```

If email must be unique:

```http
409 Conflict
```

Another common example is a state conflict such as trying to modify a resource version that has already changed.

---

### 422 Unprocessable Content

The request syntax is valid, but the submitted content fails semantic/business validation.

For example:

```json
{
  "startDate": "2026-10-10",
  "endDate": "2026-10-01"
}
```

The JSON is valid.

But the business rule is violated:

```text
startDate > endDate
```

Depending on your API conventions, this could be represented with `400` or `422`; the important thing is to define the convention consistently.

---

# 11. 5xx — Server-side problems

### 500 Internal Server Error

Something unexpected happened on the server.

```text
NullPointerException
Database failure
Unexpected exception
```

Don't expose internal details:

### ❌

```json
{
  "error": "NullPointerException at UserService.java:123"
}
```

Instead:

### ✅

```json
{
  "code": "INTERNAL_ERROR",
  "message": "An unexpected error occurred."
}
```

And log the actual exception internally.

---

# 12. Pagination

This is explicitly part of Topic 16, while the roadmap also has a dedicated Topic 17 for deeper pagination. 

Suppose we have:

```text
10 million products
```

This is obviously bad:

```http
GET /products
```

returning all 10 million.

Instead:

```http
GET /products?page=0&size=20
```

Response:

```json
{
  "data": [
    ...
  ],
  "page": 0,
  "size": 20,
  "totalElements": 10000000
}
```

We'll go much deeper into:

```text
Offset pagination
Cursor pagination
Keyset pagination
```

in Topic 17.

---

# 13. Filtering

Suppose the client wants products belonging to a particular category.

Instead of:

```text
/products/electronics
```

you could design:

```http
GET /products?category=electronics
```

Multiple filters:

```http
GET /products?category=electronics&brand=sony&minPrice=500
```

Conceptually:

```text
/products
    │
    ├── category
    ├── brand
    └── minPrice
```

The path identifies the resource.

Query parameters refine the collection.

---

# 14. Sorting

Example:

```http
GET /products?sort=price
```

Descending:

```http
GET /products?sort=-price
```

Or explicitly:

```http
GET /products?sortBy=price&sortOrder=desc
```

In a real API, I'd favor a consistent convention across the whole API rather than inventing different styles for every endpoint.

---

# 15. Filtering + pagination + sorting together

A realistic API might look like:

```http
GET /products
    ?category=electronics
    &minPrice=100
    &maxPrice=1000
    &sortBy=price
    &sortOrder=asc
    &page=0
    &size=20
```

This is a very normal REST API.

Think of it as:

```text
/products
    │
    ├── Filtering
    │     ├── category
    │     ├── minPrice
    │     └── maxPrice
    │
    ├── Sorting
    │     ├── sortBy
    │     └── sortOrder
    │
    └── Pagination
          ├── page
          └── size
```

---

# 16. API Versioning

Your roadmap lists versioning as the final part of Topic 16, with Topic 18 dedicated to going deeper. 

Suppose you currently have:

```http
GET /api/v1/users/123
```

Then you make a breaking change.

For example, old response:

```json
{
  "id": 123,
  "name": "Rohan"
}
```

New API:

```json
{
  "userId": 123,
  "firstName": "Rohan",
  "lastName": "Aggarwal"
}
```

Existing clients may break.

So you can introduce:

```http
GET /api/v2/users/123
```

while keeping:

```http
GET /api/v1/users/123
```

temporarily.

The roadmap shows exactly this path-based example. 

We'll later discuss alternatives such as header/media-type based versioning.

---

# 17. REST API Design in Spring Boot

Since you're working heavily with Spring Boot, let's map the concepts.

For example:

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    @GetMapping("/{id}")
    public UserResponse getUser(@PathVariable Long id) {
        return userService.getUser(id);
    }

    @PostMapping
    public ResponseEntity<UserResponse> createUser(
            @RequestBody CreateUserRequest request) {

        UserResponse user = userService.createUser(request);

        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(user);
    }

    @PatchMapping("/{id}")
    public UserResponse updateUser(
            @PathVariable Long id,
            @RequestBody UpdateUserRequest request) {

        return userService.updateUser(id, request);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(
            @PathVariable Long id) {

        userService.deleteUser(id);

        return ResponseEntity.noContent().build();
    }
}
```

This maps almost directly to the REST concepts:

```text
GET      → retrieve
POST     → create
PATCH    → partial update
DELETE   → delete
```

---

# 18. Don't expose your database model directly

This is a **very important system-design practice**.

Suppose your database entity is:

```java
@Entity
class User {

    Long id;
    String name;
    String passwordHash;
    String internalStatus;
    LocalDateTime createdAt;
}
```

Don't blindly return:

```java
return user;
```

because now your API representation is coupled to your database entity.

Instead:

```java
public record UserResponse(
        Long id,
        String name
) {}
```

Then:

```text
Database Entity
       ↓
    Service
       ↓
   DTO / API Model
       ↓
     Client
```

This gives you freedom to change your database without unnecessarily breaking your API contract.

---

# 19. Consistent Error Responses

This is especially relevant to your Spring Boot experience.

Instead of endpoint A returning:

```json
{
  "message": "User not found"
}
```

and endpoint B returning:

```json
{
  "error": "NOT_FOUND",
  "description": "Product doesn't exist"
}
```

define a consistent error contract.

For example:

```json
{
  "code": "USER_NOT_FOUND",
  "message": "User with id 123 was not found",
  "timestamp": "2026-10-01T15:30:00Z",
  "path": "/api/v1/users/123"
}
```

Then your global exception handling can consistently produce it.

This connects directly with the Spring Boot **global exception handling** work you've already done.

---

# 20. A complete example

Imagine we're designing an Order API.

### Create order

```http
POST /api/v1/orders
```

Request:

```json
{
  "productId": 123,
  "quantity": 2
}
```

Response:

```http
201 Created
```

```json
{
  "id": 987,
  "productId": 123,
  "quantity": 2,
  "status": "PENDING"
}
```

---

### Get order

```http
GET /api/v1/orders/987
```

```http
200 OK
```

---

### List orders

```http
GET /api/v1/orders?page=0&size=20
```

---

### Filter orders

```http
GET /api/v1/orders?status=PENDING
```

---

### Sort orders

```http
GET /api/v1/orders?sortBy=createdAt&sortOrder=desc
```

---

### Update order

```http
PATCH /api/v1/orders/987
```

```json
{
  "quantity": 3
}
```

---

### Delete order

```http
DELETE /api/v1/orders/987
```

```http
204 No Content
```

---

# 21. The mental model I want you to remember

When designing a REST API, go through these questions:

```text
1. What are my RESOURCES?
          ↓
2. How should I MODEL them?
          ↓
3. What HTTP METHOD represents the operation?
          ↓
4. What STATUS CODE represents the result?
          ↓
5. Could the collection become LARGE?
          ↓
6. If yes → PAGINATION
          ↓
7. Does the client need SEARCH/FILTERING?
          ↓
8. Does it need SORTING?
          ↓
9. Could the API contract evolve?
          ↓
10. Do I need VERSIONING?
```

That's the actual design thought process.

---

# 22. Interview example

If an interviewer says:

> **Design REST APIs for an e-commerce order system.**

Don't immediately start writing Spring controllers.

Start with resources:

```text
User
Product
Order
OrderItem
Payment
```

Then APIs:

```text
GET    /users/{id}

GET    /products
GET    /products/{id}

POST   /orders
GET    /orders/{id}
GET    /orders
PATCH  /orders/{id}
DELETE /orders/{id}

GET    /orders/{id}/items
```

Then discuss:

```text
Pagination
Filtering
Sorting
Authentication
Authorization
Error handling
Idempotency
Versioning
Rate limiting
```

And **this is where REST API design starts connecting to the rest of your system-design roadmap**.

For example:

```text
                 Client
                    │
                    ▼
              API Gateway
                    │
                    ▼
              Order Service
               /          \
              /            \
          Redis           Database
                            │
                            ▼
                          Kafka
```

REST is primarily the **client-facing communication contract**. It doesn't dictate what happens behind the API.

---

## The 7 things to remember for Topic 16

If you want the interview-ready version, remember this:

```text
REST API Design
│
├── Resource modeling
│      └── URLs represent nouns/resources
│
├── HTTP methods
│      ├── GET
│      ├── POST
│      ├── PUT
│      ├── PATCH
│      └── DELETE
│
├── Status codes
│      ├── 2xx
│      ├── 4xx
│      └── 5xx
│
├── Pagination
│
├── Filtering
│
├── Sorting
│
└── Versioning
```

And one important distinction: **Topic 16 gives you the overall REST API design foundation; Topics 17–21 then go deeper into pagination, versioning, idempotency, rate limiting, and API gateways.** 

Given your Java/Spring background, I’d pay particular attention to **resource modeling, PUT vs PATCH, status-code semantics, DTO/API contracts, and idempotency**—those are the areas where REST knowledge becomes useful in actual system-design interviews.

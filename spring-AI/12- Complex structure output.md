# Complex Structured Output — 2–3 hours

Now that you understand basic Structured Output, the next step is making the LLM return **real-world Java objects**, not just simple objects like:

```java
record Customer(String name, int age) {}
```

Real applications have structures like:

```text
Order
 ├── orderId
 ├── status
 ├── customer
 │    ├── name
 │    └── email
 └── items
      ├── Product
      ├── quantity
      └── price
```

That's what **Complex Structured Output** is about.

---

# 1. Lists

The simplest complex structure is a list.

Suppose you ask:

```text
Give me 3 Java books with their titles and authors.
```

You don't want:

```text
Book 1: Effective Java by Joshua Bloch
Book 2: Clean Code by Robert Martin
Book 3: Java Concurrency in Practice by Brian Goetz
```

You want something your Java application can consume:

```json
[
  {
    "title": "Effective Java",
    "author": "Joshua Bloch"
  },
  {
    "title": "Clean Code",
    "author": "Robert Martin"
  },
  {
    "title": "Java Concurrency in Practice",
    "author": "Brian Goetz"
  }
]
```

Your Java model could be:

```java
public record Book(
    String title,
    String author
) {}
```

And conceptually:

```java
List<Book> books;
```

The important thing here is:

```text
LLM
 ↓
JSON array
 ↓
List<Book>
```

---

# 2. Why Lists matter

A huge number of AI use cases produce collections.

For example:

### Resume parsing

```text
Resume
 ↓
List<Skill>
```

### Product extraction

```text
Invoice
 ↓
List<InvoiceItem>
```

### Recommendation system

```text
User request
 ↓
List<Product>
```

### Document analysis

```text
Document
 ↓
List<Entity>
```

So don't think of `List<T>` as a small technical feature.

It's extremely common in AI applications.

---

# 3. Nested Objects

Now let's make it more realistic.

Suppose we want an order.

```java
public record Order(
    String orderId,
    Customer customer,
    List<OrderItem> items
) {}
```

Then:

```java
public record Customer(
    String name,
    String email
) {}
```

And:

```java
public record OrderItem(
    String productName,
    int quantity,
    double price
) {}
```

Now we have:

```text
Order
│
├── orderId
│
├── customer
│   ├── name
│   └── email
│
└── items
    ├── OrderItem
    │   ├── productName
    │   ├── quantity
    │   └── price
    │
    ├── OrderItem
    │   ├── productName
    │   ├── quantity
    │   └── price
    │
    └── ...
```

The LLM should produce something like:

```json
{
  "orderId": "ORD-1001",
  "customer": {
    "name": "John",
    "email": "john@example.com"
  },
  "items": [
    {
      "productName": "iPhone",
      "quantity": 1,
      "price": 999
    },
    {
      "productName": "AirPods",
      "quantity": 2,
      "price": 199
    }
  ]
}
```

Spring AI then needs to convert this into:

```java
Order
```

with:

```java
order.customer()
```

and:

```java
order.items()
```

available as normal Java objects.

---

# 4. Lists + Nested Objects

This is where **complex structured output** really starts.

You can combine:

```text
Object
 ├── Object
 └── List<Object>
```

For example:

```java
public record Order(
    String orderId,
    Customer customer,
    List<OrderItem> items
) {}
```

This is essentially:

```text
Order
 ├── String
 ├── Customer
 │    ├── String
 │    └── String
 │
 └── List<OrderItem>
      ├── OrderItem
      ├── OrderItem
      └── OrderItem
```

This is a structure you should be comfortable designing before moving on.

---

# 5. Enums

Now comes another important concept:

**Enums.**

Suppose an order can have only these statuses:

```java
public enum OrderStatus {
    PENDING,
    CONFIRMED,
    SHIPPED,
    DELIVERED,
    CANCELLED
}
```

Your order becomes:

```java
public record Order(
    String orderId,
    OrderStatus status,
    Customer customer,
    List<OrderItem> items
) {}
```

Now the LLM should return:

```json
{
  "orderId": "ORD-1001",
  "status": "SHIPPED",
  "customer": {
    "name": "John",
    "email": "john@example.com"
  },
  "items": []
}
```

And Spring/Jackson can map:

```text
"SHIPPED"
    ↓
OrderStatus.SHIPPED
```

---

# 6. Why Enums are useful with LLMs

Without an enum, the LLM could return:

```text
"shipped"
```

or:

```text
"SHIP"
```

or:

```text
"Order has been shipped"
```

or:

```text
"shipping"
```

Now your application has to figure out what the model meant.

With an enum, you define the allowed values:

```java
PENDING
CONFIRMED
SHIPPED
DELIVERED
CANCELLED
```

So your application has a controlled vocabulary.

This is particularly useful for:

* Classification
* Status
* Priority
* Categories
* Intent detection
* Sentiment
* Routing decisions

For example:

```java
public enum Priority {
    LOW,
    MEDIUM,
    HIGH,
    CRITICAL
}
```

Then:

```java
public record SupportTicket(
    String summary,
    Priority priority
) {}
```

---

# 7. A very important issue: LLM output isn't guaranteed

Here's the mindset you need to develop.

Suppose you tell the LLM:

```text
Classify this support ticket.
```

and expect:

```java
Priority.HIGH
```

The model might produce:

```json
{
  "priority": "HIGH"
}
```

Great.

But it could also produce:

```json
{
  "priority": "high"
}
```

Or:

```json
{
  "priority": "URGENT"
}
```

Or:

```json
{
  "priority": "VERY_HIGH"
}
```

Depending on the model/provider and structured-output mechanism, invalid values may cause parsing/deserialization failures or otherwise require handling.

That's why the next topic in your roadmap is:

> **Validation**

---

# 8. Validation

Structured output answers:

> "Does the response have the expected structure?"

Validation answers:

> "Does the data actually make sense?"

These are different things.

Suppose:

```java
public record Customer(
    String name,
    int age
) {}
```

The LLM returns:

```json
{
  "name": "",
  "age": -500
}
```

The structure is correct.

You successfully have:

```java
Customer
```

But the data is obviously invalid.

So:

```text
LLM
 ↓
Structured Output
 ↓
Java Object
 ↓
Validation
 ↓
Business Logic
```

---

# 9. Bean Validation

Since you're already familiar with Spring Boot, this should feel very familiar.

You can use Jakarta Bean Validation:

```java
public record Customer(

    @NotBlank
    String name,

    @Min(0)
    @Max(150)
    int age

) {}
```

Now:

```text
"name": ""
```

fails:

```java
@NotBlank
```

And:

```text
"age": -500
```

fails:

```java
@Min(0)
```

So your normal Spring validation concepts remain useful when working with AI.

---

# 10. Nested validation

You can also validate nested objects.

For example:

```java
public record Order(
    @NotBlank
    String orderId,

    @Valid
    Customer customer,

    @NotEmpty
    List<@Valid OrderItem> items
) {}
```

This means:

```text
Order
 │
 ├── orderId
 │     ↓
 │   validate
 │
 ├── customer
 │     ↓
 │   @Valid
 │     ↓
 │   validate Customer
 │
 └── items
       ↓
     @NotEmpty
       +
     @Valid each item
```

This is very similar to normal Spring Boot DTO validation.

---

# 11. Validation is NOT prompt engineering

This distinction is important.

You might tell the LLM:

```text
Age must be between 0 and 150.
```

That's useful.

But don't rely solely on that.

Think:

```text
Prompt:
"Please return a valid age between 0 and 150."

             +

Java validation:
@Min(0)
@Max(150)
```

The prompt guides the model.

The application validates the result.

**Your Java application remains the final authority.**

---

# 12. Error Handling

Now imagine the LLM produces something that cannot be converted to your Java object.

For example:

```json
{
  "orderId": "ORD-1001",
  "status": "UNKNOWN_STATUS"
}
```

but your enum is:

```java
public enum OrderStatus {
    PENDING,
    CONFIRMED,
    SHIPPED,
    DELIVERED,
    CANCELLED
}
```

You may get a conversion/deserialization error.

Your application shouldn't simply crash.

You need a strategy.

---

# 13. Typical failure pipeline

Think of your AI call like this:

```text
                 LLM
                  │
                  ▼
          Structured Response
                  │
                  ▼
            Deserialization
                  │
            ┌─────┴─────┐
            │           │
          SUCCESS      ERROR
            │           │
            ▼           ▼
       Java Object   Error Handling
            │
            ▼
        Validation
            │
       ┌────┴────┐
       │         │
     VALID     INVALID
       │         │
       ▼         ▼
 Business     Validation
 Logic         Error
```

This is a much better mental model than:

```text
LLM → Java Object → done
```

---

# 14. What should you do when parsing fails?

For example:

```text
LLM
 ↓
Invalid structured response
 ↓
Parsing exception
```

You could:

### Option 1 — Retry

Ask the model again.

```text
Attempt 1
   ↓
Invalid
   ↓
Retry
   ↓
Attempt 2
```

Useful for transient/model-format failures.

But don't retry infinitely. 😄

Usually you would have a small retry limit.

---

### Option 2 — Return an application error

For example:

```text
Unable to process the request.
Please try again.
```

---

### Option 3 — Log and investigate

In production, you should log enough information to understand:

* Which model was used
* Which operation failed
* Whether parsing failed
* Whether validation failed
* Relevant request/correlation ID

Be careful about logging sensitive user data.

---

# 15. Parsing error vs Validation error

This distinction is **very important**.

### Parsing/deserialization error

The response cannot be converted into your Java object.

Example:

```json
{
  "age": "hello"
}
```

when you expect:

```java
int age
```

Think:

> "I cannot understand this response as my expected Java structure."

---

### Validation error

You successfully created the object, but its values aren't acceptable.

Example:

```json
{
  "age": -100
}
```

becomes:

```java
Customer(age=-100)
```

but:

```java
@Min(0)
```

fails.

Think:

> "I understood the response, but the data isn't acceptable."

---

# 16. Complex example

Let's combine everything.

Imagine an AI support-ticket extractor.

### Enums

```java
public enum Category {
    BILLING,
    TECHNICAL,
    ACCOUNT,
    GENERAL
}
```

```java
public enum Priority {
    LOW,
    MEDIUM,
    HIGH,
    CRITICAL
}
```

### Customer

```java
public record Customer(
    @NotBlank String name,
    @Email String email
) {}
```

### Ticket

```java
public record SupportTicket(
    @NotBlank String summary,

    Category category,

    Priority priority,

    @Valid
    Customer customer,

    @NotEmpty
    List<@NotBlank String> issues
) {}
```

Now the user says:

```text
Hi, I'm John.
My email is john@example.com.

I cannot log into my account and I have
already tried resetting my password twice.
```

The LLM should produce something like:

```json
{
  "summary": "Customer cannot log into account after trying password reset",
  "category": "ACCOUNT",
  "priority": "HIGH",
  "customer": {
    "name": "John",
    "email": "john@example.com"
  },
  "issues": [
    "Cannot log into account",
    "Password reset unsuccessful"
  ]
}
```

Spring AI converts it into:

```text
SupportTicket
 ├── summary
 ├── category → ACCOUNT
 ├── priority → HIGH
 ├── customer
 │    ├── name
 │    └── email
 │
 └── issues
      ├── issue 1
      └── issue 2
```

Then validation happens.

Only after validation passes should your business logic process the ticket.

---

# 17. The architecture you should remember

For complex structured output, remember this pipeline:

```text
                       User Input
                           │
                           ▼
                     Prompt + Schema
                           │
                           ▼
                          LLM
                           │
                           ▼
                  Structured Response
                           │
                           ▼
                    Deserialization
                           │
                    ┌──────┴──────┐
                    │             │
                  Success        Error
                    │             │
                    ▼             ▼
              Java Object     Retry / Error
                    │
                    ▼
                Validation
                    │
              ┌─────┴─────┐
              │           │
            Valid       Invalid
              │           │
              ▼           ▼
        Business Logic   Error
```

This is probably the **most important diagram of this topic**.

---

# 18. How this differs from simple Structured Output

You previously learned:

```java
public record Product(
    String name,
    double price
) {}
```

That's basic structured output.

Now you're moving toward:

```java
public record Order(
    String orderId,
    OrderStatus status,
    Customer customer,
    List<OrderItem> items
) {}
```

which contains:

```text
✓ Simple fields
✓ Nested objects
✓ Lists
✓ Enums
✓ Validation
✓ Error handling
```

That's **complex structured output**.

---

# 19. What you should practice during your 2–3 hours

I'd actually do **one project**, rather than five unrelated examples.

Build an **AI Support Ticket Extractor**.

### Exercise 1 — Simple object

Input:

```text
My application is crashing.
```

Output:

```java
SupportTicket
```

with:

```text
summary
```

---

### Exercise 2 — Add enum

Add:

```java
Category
```

with:

```text
BILLING
TECHNICAL
ACCOUNT
GENERAL
```

---

### Exercise 3 — Add another enum

Add:

```java
Priority
```

with:

```text
LOW
MEDIUM
HIGH
CRITICAL
```

---

### Exercise 4 — Add nested object

Add:

```java
Customer
```

inside:

```java
SupportTicket
```

---

### Exercise 5 — Add list

Add:

```java
List<String> issues
```

---

### Exercise 6 — Add validation

Add:

```java
@NotBlank
@Email
@NotEmpty
```

and test invalid results.

---

### Exercise 7 — Error handling

Intentionally create situations where:

```text
LLM output
   ↓
cannot be converted
```

and:

```text
LLM output
   ↓
converted successfully
   ↓
fails validation
```

Handle both separately.

---

# 20. Your key takeaway

You can think of the entire topic as:

```text
Basic Structured Output
        ↓
       Object
        ↓
Complex Structured Output
        ↓
 ┌──────┼───────────┐
 ↓      ↓           ↓
Lists  Nested     Enums
       Objects
        ↓
    Validation
        ↓
  Error Handling
```

And the **production mindset** is:

> **Never trust an LLM response just because it successfully parsed into a Java object.**

Parsing tells you **"this has the expected shape."**

Validation tells you **"this data is acceptable."**

Business logic tells you **"this is what my application should do."**

That's the real difference between a simple Spring AI demo and a production-quality AI feature.

# 39. Tool / Function Calling

## 1. The problem

Normally, an LLM can only **generate a response**.

For example:

```text
User:
What is the weather in Abu Dhabi?

        ↓

LLM

        ↓

"I don't have access to current weather information."
```

The LLM itself doesn't magically have access to your database, REST APIs, Java services, etc.

Tool calling solves this.

---

# 2. What is a Tool?

A **tool is a function that your application makes available to the LLM**.

For example:

```java
getWeather(String city)
```

or:

```java
getCustomer(Long customerId)
```

or:

```java
getOrder(Long orderId)
```

The tool performs the actual operation.

Think of it like:

```text
LLM
 │
 │ decides which tool to use
 ↓
Java Method
 │
 │ executes real business logic
 ↓
Result
 │
 ↓
LLM
 │
 ↓
Natural language answer
```

This is exactly the flow represented in your roadmap. 

---

# 3. Important point: the LLM does NOT execute your Java method

This is probably the most important concept to understand.

Suppose you have:

```java
getWeather("Abu Dhabi")
```

The LLM doesn't actually execute this Java method.

Instead, the process is approximately:

```text
User
 │
 │ "What's the weather in Abu Dhabi?"
 ↓
LLM
 │
 │ decides:
 │ "I need getWeather"
 │
 │ arguments:
 │ city = "Abu Dhabi"
 ↓
Spring AI
 │
 │ invokes Java method
 ↓
getWeather("Abu Dhabi")
 │
 ↓
Weather API
 │
 ↓
"32°C, Sunny"
 │
 ↓
Spring AI
 │
 │ gives tool result back to LLM
 ↓
LLM
 │
 ↓
"It's currently 32°C and sunny in Abu Dhabi."
```

So there are **two different responsibilities**:

### LLM

Decides:

> "I need weather information, so I should call `getWeather`."

### Your application

Actually performs:

```java
getWeather("Abu Dhabi")
```

That's the key distinction.

---

# 4. Why is this called "function calling"?

Because conceptually the LLM produces something like:

```json
{
  "name": "getWeather",
  "arguments": {
    "city": "Abu Dhabi"
  }
}
```

Your application sees that and invokes:

```java
getWeather("Abu Dhabi");
```

The exact wire format depends on the model/provider, but conceptually this is what happens.

---

# 5. Tool calling vs normal prompting

Without tools:

```text
User
 ↓
Prompt
 ↓
LLM
 ↓
Text
```

With tools:

```text
User
 ↓
LLM
 ↓
Tool decision
 ↓
Java method
 ↓
Tool result
 ↓
LLM
 ↓
Final answer
```

This is why tool calling is such a big step in your Spring AI roadmap.

The LLM goes from being a:

> **text generator**

to being a:

> **reasoning/decision component that can interact with application capabilities.**

---

# 6. A realistic example

Imagine an e-commerce application.

User says:

```text
Where is my order 12345?
```

Your application exposes:

```java
getOrderStatus(12345)
```

The LLM may determine:

```text
I need order information.

Tool:
getOrderStatus

Arguments:
orderId = 12345
```

Your Java application executes:

```java
OrderStatus status = orderService.getOrderStatus(12345);
```

Result:

```json
{
    "orderId": 12345,
    "status": "SHIPPED",
    "expectedDelivery": "2026-09-28"
}
```

The result goes back to the LLM.

Then the LLM produces:

```text
Your order 12345 has been shipped and is expected
to arrive on September 28.
```

---

# 7. Tool calling is not the same as RAG

This is especially important because you've already studied RAG.

### RAG

RAG answers questions using **retrieved knowledge**.

```text
Question
   ↓
Embedding
   ↓
Vector Search
   ↓
Relevant Documents
   ↓
LLM
   ↓
Answer
```

Example:

> "What is our company's maternity leave policy?"

Search your documents.

---

### Tool calling

Tool calling allows the LLM to **perform an operation or retrieve live information**.

```text
Question
   ↓
LLM
   ↓
Tool
   ↓
Database/API/Service
   ↓
Result
   ↓
LLM
   ↓
Answer
```

Example:

> "What's the status of my order?"

Call your order service.

---

### Together

Real AI applications often use both:

```text
                     ┌── RAG ──→ Company documents
                     │
User → LLM → Decision
                     │
                     └── Tool ─→ Database/API
```

That's one reason your roadmap intentionally teaches RAG, Memory, and Tool Calling before Agents.

---

# 40. Creating Tools

Now we move from:

> "What is tool calling?"

to:

> "How do I expose my Java functionality as a tool?"

Your roadmap suggests examples such as `getCustomer()`, `getOrder()`, `getWeather()`, `searchProducts()`, and `calculateTax()`. 

In Spring AI, you can create tools around Java methods.

A simplified example looks like this:

```java
public class WeatherTools {

    @Tool(description = "Get the current weather for a city")
    public String getWeather(String city) {

        // Call weather API
        return "32°C and Sunny";
    }
}
```

The important part is:

```java
@Tool
```

You're telling Spring AI:

> "This Java method is available as a tool that an AI model can call."

---

# 8. Tool description is extremely important

Consider:

```java
@Tool
public String getWeather(String city) {
    ...
}
```

That's not very descriptive.

Instead:

```java
@Tool(
    description = "Get the current weather for a given city"
)
public String getWeather(String city) {
    ...
}
```

Why?

Because the LLM needs to understand **when this tool should be used**.

Imagine you expose:

```java
getCustomer()
getOrder()
getProduct()
getWeather()
```

The model needs enough information to distinguish them.

Think of the tool metadata as the LLM's **API documentation**.

---

# 9. Tool parameters

You can also have multiple parameters.

For example:

```java
@Tool(description = "Get an order for a customer")
public Order getOrder(
        Long customerId,
        Long orderId) {

    return orderService.getOrder(customerId, orderId);
}
```

The model can determine:

```text
Tool:
getOrder

Arguments:
customerId = 100
orderId = 5001
```

Spring AI handles the tool invocation mechanism.

---

# 10. Registering the tool with ChatClient

Conceptually you can make the tool available to a `ChatClient`.

For example:

```java
chatClient.prompt()
        .user("Where is order 5001?")
        .tools(orderTools)
        .call()
        .content();
```

The important idea is:

```text
ChatClient
    │
    ├── Prompt
    │
    └── Tools
         │
         ├── getOrder()
         ├── getCustomer()
         └── getWeather()
```

The model receives the descriptions of the available tools and can decide whether one is needed.

---

# 11. Complete example

Let's create a small customer tool.

### Tool

```java
@Component
public class CustomerTools {

    private final CustomerService customerService;

    public CustomerTools(CustomerService customerService) {
        this.customerService = customerService;
    }

    @Tool(description = "Find a customer using their customer ID")
    public Customer getCustomer(Long customerId) {

        return customerService.findById(customerId);
    }
}
```

Then:

```java
String response = chatClient.prompt()
        .user("Tell me the details of customer 101")
        .tools(customerTools)
        .call()
        .content();
```

The conceptual flow is:

```text
User
 │
 │ "Tell me details of customer 101"
 ↓
ChatClient
 ↓
LLM
 │
 │ Tool required
 │
 │ getCustomer(101)
 ↓
Spring AI
 ↓
CustomerTools
 ↓
customerService.findById(101)
 ↓
Database
 ↓
Customer
 ↓
LLM
 ↓
Final response
```

---

# 12. Tool calling can involve multiple rounds

This is another important concept.

Suppose the user says:

> "Tell me whether customer 101's order is delayed."

The LLM may need multiple tools:

```text
LLM
 │
 ├── getCustomer(101)
 │
 ↓
Customer information
 │
 ↓
LLM
 │
 ├── getOrder(customerId=101)
 │
 ↓
Order
 │
 ↓
LLM
 │
 ├── getDeliveryStatus(orderId=500)
 │
 ↓
Delivery status
 │
 ↓
LLM
 │
 ↓
Final answer
```

This is moving toward the **Agent** concepts you'll study later.

Your roadmap explicitly puts **Multiple tools** at topic 41 and **Tool security** at topic 42. 

---

# 13. Tool calling vs REST API

You may be thinking:

> "This sounds like calling REST APIs. What's new?"

The difference is **who decides what to call**.

Traditional application:

```text
User
 ↓
Backend code
 ↓
if request == weather:
    weatherService.getWeather()
else if request == order:
    orderService.getOrder()
```

The developer defines the decision logic.

With tool calling:

```text
User
 ↓
LLM
 ↓
"Which capability do I need?"
 ↓
Tool
```

The LLM determines which available capability is relevant.

That's the magic.

---

# 14. But don't give the LLM unrestricted access

This is where your backend/security experience becomes very important.

Imagine exposing:

```java
deleteCustomer()
transferMoney()
changePassword()
executeSql()
```

as tools.

You should **not** blindly trust the LLM.

Your roadmap specifically calls out:

```text
Authorization
Validation
Malicious parameters
Tool abuse
Maximum execution limits
```

for topic 42. 

Think of a tool as an API endpoint.

You would never do this:

```java
@PostMapping("/deleteCustomer")
public void delete(...) {
    // Trust whatever the client sends
}
```

Similarly, don't do:

```text
LLM → unrestricted business operation
```

Instead:

```text
LLM
 ↓
Tool
 ↓
Authentication / Authorization
 ↓
Input validation
 ↓
Business rules
 ↓
Operation
```

---

# 15. The mental model I want you to remember

Since you're learning this from an Architect/Principal Engineer perspective, I'd remember tool calling as this:

```text
                 ┌──────────────────┐
                 │       User       │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │       LLM        │
                 │                  │
                 │ "What do I need?"│
                 └────────┬─────────┘
                          ↓
                  Tool selection
                          ↓
                 ┌──────────────────┐
                 │   Spring AI      │
                 │  Tool Framework  │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │    Java Tool     │
                 └────────┬─────────┘
                          ↓
              ┌───────────┼───────────┐
              ↓           ↓           ↓
            DB          REST API    Service
              │           │           │
              └───────────┼───────────┘
                          ↓
                    Tool Result
                          ↓
                         LLM
                          ↓
                    Final Answer
```

## 39 vs 40

| Topic                         | What you learn                                         |
| ----------------------------- | ------------------------------------------------------ |
| **39. Tool/function calling** | How an LLM decides to use an external capability       |
| **40. Creating tools**        | How you expose your Java methods as those capabilities |

So, **39 is the architecture/concept**, while **40 is the Spring AI implementation**.

And the progression in your roadmap makes sense:

```text
RAG
 ↓
Memory
 ↓
39. Tool Calling
 ↓
40. Creating Tools
 ↓
41. Multiple Tools
 ↓
42. Tool Security
 ↓
MCP
 ↓
Agents
```

That sequence takes you from **"LLM can retrieve information" → "LLM can interact with my application" → "LLM can use multiple capabilities" → "LLM can operate safely and autonomously."** 

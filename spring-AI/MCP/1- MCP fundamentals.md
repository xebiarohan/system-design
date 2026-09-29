# MCP Fundamentals

## 1. First: What problem does MCP solve?

You already learned Tool Calling:

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
Answer
```

For example, your Spring AI application might have:

```java
@Tool
public Customer getCustomer(Long id) {
    return customerService.findById(id);
}
```

The LLM can decide:

> "I need customer information, so I'll call `getCustomer`."

This works well.

But imagine your company has **20 different AI applications**:

```text
AI Application 1 ──→ Customer APIs
AI Application 2 ──→ Customer APIs
AI Application 3 ──→ Customer APIs
AI Application 4 ──→ Customer APIs
```

And each application needs:

* Customer tools
* Order tools
* Database tools
* File-system access
* Git tools
* Search tools
* Internal APIs

Without a common protocol, every AI application has to integrate with every tool provider independently.

That creates a lot of integration code.

MCP addresses this problem.

---

# 2. What is MCP?

**MCP = Model Context Protocol**

Think of MCP as a **standard protocol for connecting AI applications to external capabilities and context**.

The key idea:

```text
              MCP
               ↓
      Standard communication
               ↓
       AI Application
          ↙    ↓    ↘
       Tools Resources Prompts
```

Instead of an AI application having custom integration logic for every external system, it can communicate with an **MCP server** using the MCP protocol.

Conceptually:

```text
Without MCP

AI App
 ├── custom Customer integration
 ├── custom Git integration
 ├── custom Database integration
 ├── custom Search integration
 └── custom File integration
```

With MCP:

```text
AI App
   ↓
MCP Client
   ↓
MCP Protocol
   ↓
MCP Servers
   ├── Customer MCP Server
   ├── Git MCP Server
   ├── Database MCP Server
   ├── Search MCP Server
   └── File MCP Server
```

This is the fundamental reason MCP is useful.

---

# 3. MCP is a protocol, not an AI model

This distinction is important.

MCP is **not**:

* an LLM
* an AI model
* a vector database
* a replacement for RAG
* a replacement for Spring AI
* a tool itself

Instead:

```text
LLM
 ↑
Spring AI
 ↑
MCP Client
 ↑
MCP Protocol
 ↑
MCP Server
 ↑
External system
```

So MCP sits in the **integration layer**.

---

# 4. MCP Client vs MCP Server

This is probably the most important concept in this topic.

## MCP Client

The **MCP Client** is part of the AI application.

Its job is to communicate with MCP servers.

For example:

```text
Spring Boot AI Application
          │
          │
      MCP Client
          │
          ↓
     MCP Server
```

The client can ask the MCP server:

> What tools do you provide?

or:

> Give me this resource.

or:

> Execute this tool.

---

# 5. MCP Server

The **MCP Server** exposes capabilities to MCP clients.

For example:

```text
Customer MCP Server
       │
       ├── getCustomer()
       ├── getOrders()
       └── searchCustomers()
```

Another server:

```text
Git MCP Server
       │
       ├── getRepository()
       ├── searchCode()
       └── createBranch()
```

Another:

```text
Database MCP Server
       │
       ├── queryDatabase()
       └── getSchema()
```

The MCP server is responsible for communicating with the underlying system.

For example:

```text
MCP Server
    ↓
Customer Service
    ↓
Database
```

The AI application doesn't necessarily need to know how that underlying system works.

---

# 6. A useful mental model

Think of MCP like **USB for AI applications**.

USB provides a standard way for a computer to communicate with different devices.

You don't need a completely different protocol for:

```text
Keyboard
Mouse
Camera
Printer
```

Similarly, MCP provides a standardized way for AI applications to interact with different external capabilities.

Conceptually:

```text
AI Application
      │
      │ MCP
      │
 ┌────┴─────────────┐
 ↓                  ↓
MCP Server       MCP Server
Customer         Git
 ↓                  ↓
Customer API     Git Repository
```

That's the architectural idea you want to remember.

---

# 7. MCP has three important concepts

Your roadmap explicitly lists:

```text
Tools
Resources
Prompts
```

These are different things.

---

# 8. MCP Tools

You've already learned Tool Calling, so this should feel familiar.

An MCP **Tool** represents an operation that the AI can invoke.

For example:

```text
getCustomer
```

Input:

```json
{
  "customerId": 123
}
```

Output:

```json
{
  "id": 123,
  "name": "John",
  "email": "john@example.com"
}
```

Another tool:

```text
getWeather
```

Another:

```text
searchProducts
```

Another:

```text
createOrder
```

The important idea:

> **Tools perform actions.**

Think:

```text
Tool → "Do something"
```

Examples:

```text
getCustomer()
createOrder()
searchProducts()
sendEmail()
queryDatabase()
```

---

# 9. MCP Resources

Resources are different.

A **resource represents information/context that can be read**.

Think:

```text
Resource → "Give me information"
```

For example, an MCP server might expose:

```text
file:///project/README.md
```

or:

```text
database://customers/schema
```

or:

```text
git://repository/main/README
```

The AI application can access these resources through MCP.

Conceptually:

```text
MCP Server
   │
   ├── Tools
   │     ├── searchCode()
   │     └── createBranch()
   │
   └── Resources
         ├── repository README
         ├── source files
         └── documentation
```

### Tool vs Resource

This distinction is worth memorizing:

|         | Tool               | Resource            |
| ------- | ------------------ | ------------------- |
| Purpose | Perform operation  | Provide information |
| Nature  | Action             | Context/data        |
| Example | `createOrder()`    | `order://123`       |
| Example | `searchProducts()` | Product catalog     |
| Think   | **Do**             | **Read**            |

---

# 10. MCP Prompts

MCP also defines **Prompts**.

A prompt is a reusable prompt/template that an MCP server can expose.

For example, suppose your company has a standard code-review prompt:

```text
Review this Java code.

Check:
1. Security
2. Performance
3. Error handling
4. Maintainability
5. Testing
```

Instead of every AI application implementing that prompt itself, an MCP server could expose it as a reusable prompt.

Conceptually:

```text
MCP Server
   │
   ├── Tools
   ├── Resources
   └── Prompts
```

Think:

```text
Tool     → Do something
Resource → Give me information
Prompt   → Give me a reusable instruction/template
```

---

# 11. Putting everything together

Suppose we build an **MCP server for an e-commerce system**.

It could expose:

```text
E-Commerce MCP Server

Tools
 ├── getCustomer()
 ├── searchProducts()
 ├── createOrder()
 └── cancelOrder()

Resources
 ├── product://catalog
 ├── customer://schema
 └── order://schema

Prompts
 ├── product-recommendation
 └── customer-support
```

Then your Spring AI application connects to it:

```text
                    Spring AI Application
                           │
                           ↓
                      MCP Client
                           │
                    MCP Protocol
                           │
                           ↓
                  E-Commerce MCP Server
                    ┌──────┼──────┐
                    ↓      ↓      ↓
                  Tools Resources Prompts
                    │
                    ↓
             E-Commerce System
```

---

# 12. How is this different from the Tool Calling you just learned?

This is **the key question for you**.

You just completed:

```text
Spring AI Tool Calling
```

You might ask:

> "Why do I need MCP? I already have tools."

Excellent question.

### Traditional Spring AI Tool Calling

Your tools can live directly inside your application:

```text
Spring Boot
    │
    ├── CustomerService
    ├── OrderService
    ├── WeatherService
    └── ProductService
```

Your application owns the tools.

---

### MCP

The tools can live behind an MCP server:

```text
Spring Boot
     │
 MCP Client
     │
     ↓
 MCP Server
     │
     ├── Customer
     ├── Orders
     ├── Products
     └── Weather
```

This allows the tool provider to be separated from the AI application.

That's the architectural shift.

---

# 13. Tool Calling vs MCP

Don't think:

```text
Tool Calling OR MCP
```

Think:

```text
Tool Calling
     +
MCP
```

They solve related but different problems.

**Tool calling** answers:

> How can an LLM request that an operation be executed?

**MCP** answers more broadly:

> How can an AI application discover and interact with external tools, resources, and prompts through a standardized protocol?

So:

```text
LLM
 ↓
Tool Calling
 ↓
MCP Client
 ↓
MCP Protocol
 ↓
MCP Server
 ↓
Tool
 ↓
External System
```

This is a very useful architecture to keep in your head.

---

# 14. Real-world example

Imagine you build:

**Customer Support AI**

Your Spring Boot application receives:

> "Where is my order #12345?"

The architecture could be:

```text
User
 │
 ↓
React
 │
 ↓
Spring Boot
 │
 ↓
Spring AI
 │
 ↓
LLM
 │
 │ "I need order information"
 ↓
MCP Client
 │
 ↓
Order MCP Server
 │
 ↓
getOrder(12345)
 │
 ↓
Order Service
 │
 ↓
Order information
 │
 ↓
MCP Client
 │
 ↓
LLM
 │
 ↓
"Your order is currently shipped."
```

Notice something important:

The LLM doesn't directly connect to your Order Service.

Instead:

```text
LLM
 ↓
AI Application
 ↓
MCP Client
 ↓
MCP Server
 ↓
Order Service
```

That separation is one of the major architectural benefits.

---

# 15. Why MCP becomes particularly useful at scale

Imagine you have:

```text
10 AI applications
```

and:

```text
50 external capabilities
```

Without a standard protocol, you can end up with many custom integrations.

With MCP, you can build reusable MCP servers:

```text
                 MCP Servers

             ┌── Customer
             ├── Orders
AI Apps ─────┼── Git
             ├── Jira
             ├── Database
             └── Documentation
```

Multiple AI applications can potentially consume those MCP servers.

That gives you a much cleaner architecture:

```text
                  AI App A
                     │
                  MCP Client
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
     Customer       Git        Database
       MCP          MCP           MCP
      Server       Server        Server
        ↑            ↑            ↑
        │            │            │
     AI App B     AI App C     AI App D
```

---

# 16. One important security point

MCP does **not** magically make tools safe.

Remember what you learned in Topic 42:

```text
Tool security
 ├── Authorization
 ├── Validation
 ├── Malicious parameters
 ├── Tool abuse
 └── Execution limits
```

Those concerns still exist with MCP.

In fact, once you expose tools through an MCP server, you need to think carefully about:

```text
Who can connect?
       ↓
Which tools can they discover?
       ↓
Which tools can they execute?
       ↓
What parameters are allowed?
       ↓
What systems can those tools access?
```

For example, exposing:

```text
deleteCustomer()
```

is very different from exposing:

```text
getCustomer()
```

So when you reach **AI Safety** later in the roadmap, MCP + Tool Security will connect nicely.

---

# 17. MCP architecture — your mental picture

For this topic, I'd memorize this:

```text
                         AI Application
                               │
                               ↓
                         MCP Client
                               │
                         MCP Protocol
                               │
                               ↓
                         MCP Server
                      ┌────────┼────────┐
                      ↓        ↓        ↓
                    Tools   Resources Prompts
                      │        │        │
                      ↓        ↓        ↓
                 External systems / data
```

And remember:

```text
Tool
→ Do something

Resource
→ Provide information

Prompt
→ Provide reusable instructions
```

---

# 18. MCP vs RAG vs Tool Calling

Since you've already learned all three, this comparison is particularly useful:

| Concept          | Main purpose                                             |
| ---------------- | -------------------------------------------------------- |
| **RAG**          | Retrieve relevant knowledge                              |
| **Tool Calling** | Allow LLM to request an operation                        |
| **MCP**          | Standardize AI ↔ external capability/context integration |
| **Agent**        | Reason + plan + repeatedly use tools/context             |

A simplified architecture could therefore become:

```text
                     User
                       ↓
                      LLM
                       │
              ┌────────┼────────┐
              ↓        ↓        ↓
             RAG    Tool Call   MCP
              │        │        │
          pgvector   Java Tool  MCP Server
                                │
                         ┌──────┼──────┐
                         ↓      ↓      ↓
                       Tools Resources Prompts
```

And later, in **Phase 14 Agents**, the agent can orchestrate these capabilities:

```text
User Goal
    ↓
  Agent
    ↓
  Plan
    ↓
 ┌──┴─────────────┐
 ↓                ↓
RAG              MCP
                  ↓
                Tools
                  ↓
               External
               systems
```

The next topic in your roadmap, **MCP Architecture**, will make this much more concrete: we'll look at the actual communication flow between **Spring AI application → MCP Client → MCP Server → Tools/Resources**, including what happens when a tool is invoked.

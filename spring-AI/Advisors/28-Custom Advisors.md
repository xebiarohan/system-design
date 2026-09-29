
# Custom Advisors

## 1. Why do we need Custom Advisors?

Built-in Advisors handle common Spring AI concerns:

```text
Memory
Retrieval
Question/Answer
```

But your application will eventually have its own requirements.

For example:

```text
Every AI request must:
    ↓
check user authorization
    ↓
add tenant information
    ↓
log request
    ↓
measure execution time
    ↓
apply company-specific instructions
    ↓
call LLM
```

You don't want this logic duplicated in every service.

Without an Advisor:

```java
public String ask(String question) {

    checkSecurity();

    addTenantContext();

    logRequest(question);

    String response = chatClient
            .prompt()
            .user(question)
            .call()
            .content();

    logResponse(response);

    return response;
}
```

Then another service does the same thing:

```java
public String summarize(String text) {

    checkSecurity();

    addTenantContext();

    logRequest(text);

    // ...

}
```

That's a classic cross-cutting concern.

A custom Advisor lets you encapsulate that behavior.

---

# 2. The core mental model

Think of a custom Advisor as:

```text
                    Custom Advisor
                         │
                         ↓
AI Request ──────→ [modify/intercept]
                         │
                         ↓
                      next()
                         │
                         ↓
                       LLM
                         │
                         ↓
                 [modify response]
                         │
                         ↓
                    AI Response
```

The important word here is **chain**.

An Advisor doesn't necessarily terminate the request.

It can:

1. inspect the request
2. modify the request
3. pass it to the next component
4. inspect the response
5. modify the response
6. return it

---

# 3. Think about it like Spring middleware

Because you have strong Spring/backend experience, here's the easiest analogy.

You might already know:

```text
HTTP Request
     ↓
Filter
     ↓
Interceptor
     ↓
Controller
     ↓
Service
```

A custom Advisor provides a similar compositional idea around an AI interaction:

```text
ChatClient Request
       ↓
Advisor
       ↓
Advisor
       ↓
Chat Model
       ↓
Advisor
       ↓
Response
```

So you can think of:

> **Custom Advisor = middleware for an AI request/response pipeline.**

It's not literally a Servlet Filter or Spring MVC interceptor, but the architectural analogy is very useful.

---

# 4. Advisor chain

This is the most important concept.

Suppose you have three custom Advisors:

```text
SecurityAdvisor
LoggingAdvisor
CompanyPolicyAdvisor
```

You could have:

```text
User
 ↓
ChatClient
 ↓
SecurityAdvisor
 ↓
LoggingAdvisor
 ↓
CompanyPolicyAdvisor
 ↓
LLM
 ↓
CompanyPolicyAdvisor
 ↓
LoggingAdvisor
 ↓
SecurityAdvisor
 ↓
Response
```

Conceptually, each Advisor wraps the next operation.

This resembles:

```text
A(
   B(
      C(
         LLM()
      )
   )
)
```

That's a very useful mental model.

---

# 5. The actual Spring AI abstraction

Spring AI provides Advisor abstractions for this.

The important concepts you'll encounter are:

```java
Advisor
```

and, depending on the type of advisor you're implementing:

```java
CallAdvisor
StreamAdvisor
```

The exact APIs can vary between Spring AI versions, so when implementing this in your current project, use the interfaces/method signatures provided by your version.

Conceptually, however, the job remains:

```text
Request
   ↓
Advisor
   ↓
next component
   ↓
Response
```

---

# 6. Example: Logging Advisor

Let's build one.

Requirement:

> Every AI request and response should be logged.

Without a custom Advisor:

```java
String question = "...";

log.info("AI request: {}", question);

String response = chatClient
        .prompt()
        .user(question)
        .call()
        .content();

log.info("AI response: {}", response);
```

Now imagine 20 different places calling the LLM.

That's repetitive.

Instead:

```text
ChatClient
    ↓
LoggingAdvisor
    ↓
LLM
```

Then every request automatically passes through the logging logic.

---

# 7. Conceptual implementation

The implementation pattern looks roughly like:

```java
public class LoggingAdvisor implements CallAdvisor {

    @Override
    public ChatClientResponse adviseCall(
            ChatClientRequest request,
            CallAdvisorChain chain) {

        log.info("AI request: {}", request);

        ChatClientResponse response =
                chain.nextCall(request);

        log.info("AI response: {}", response);

        return response;
    }
}
```

Don't focus too much on memorizing the exact method signature yet.

Focus on this:

```java
before:

    log(request)

    ↓

chain.nextCall(request)

    ↓

after:

    log(response)
```

That's the heart of a custom Advisor.

---

# 8. What is `chain.nextCall()`?

This is **very important**.

Suppose you have:

```text
LoggingAdvisor
       ↓
SecurityAdvisor
       ↓
LLM
```

When `LoggingAdvisor` executes:

```java
chain.nextCall(request);
```

it effectively means:

> "I'm done with my part. Continue processing the request."

So:

```text
LoggingAdvisor
      |
      | nextCall()
      ↓
SecurityAdvisor
      |
      | nextCall()
      ↓
LLM
```

Eventually the response comes back:

```text
LLM
 ↓
SecurityAdvisor
 ↓
LoggingAdvisor
 ↓
Client
```

This is why Advisors can be chained.

---

# 9. Example: Security Advisor

Now let's build something more interesting.

Imagine your AI application requires:

```text
Only users with AI_ASSISTANT permission
can invoke the LLM.
```

You could create:

```java
public class SecurityAdvisor implements CallAdvisor {

    @Override
    public ChatClientResponse adviseCall(
            ChatClientRequest request,
            CallAdvisorChain chain) {

        if (!hasPermission()) {
            throw new AccessDeniedException(
                "User is not allowed to use AI"
            );
        }

        return chain.nextCall(request);
    }
}
```

The flow becomes:

```text
User
 ↓
SecurityAdvisor
 ↓
Authorized?
 ├── NO → reject
 │
 └── YES
      ↓
     LLM
```

This is a great example of why Advisors are useful.

You don't need to put:

```java
if (!hasPermission())
```

inside every AI service.

---

# 10. Example: Request modification

Advisors don't have to only log or reject.

They can **modify the request**.

Imagine your company wants every AI request to include:

```text
You are an internal company assistant.

Never reveal confidential company information.
```

You could have:

```text
User question
     ↓
CompanyPolicyAdvisor
     ↓
Add system instruction
     ↓
LLM
```

Conceptually:

```java
ChatClientRequest modifiedRequest =
        addCompanyPolicy(request);

return chain.nextCall(modifiedRequest);
```

So:

```text
Original Request
       ↓
Advisor
       ↓
Modified Request
       ↓
Next Advisor / LLM
```

This is one of the most powerful uses of Advisors.

---

# 11. Example: Tenant Advisor

This becomes particularly interesting for a SaaS application.

Suppose:

```text
Tenant A
Tenant B
Tenant C
```

You don't want a request from Tenant A accidentally retrieving Tenant B's documents.

You could have:

```text
TenantAdvisor
      ↓
tenantId = tenant-123
      ↓
Retrieval Advisor
      ↓
Vector Store
```

The retrieval operation can then use:

```text
tenantId = tenant-123
```

as metadata filtering.

Conceptually:

```text
User Question
     ↓
Tenant Advisor
     ↓
Add tenant context
     ↓
Retrieval Advisor
     ↓
Vector DB
     ↓
Only tenant-123 documents
```

That's a much more architecture-oriented use of Advisors.

---

# 12. Example: Timing Advisor

Another very practical Advisor:

```text
Start timer
     ↓
LLM call
     ↓
Stop timer
     ↓
Record latency
```

Conceptually:

```java
long start = System.currentTimeMillis();

ChatClientResponse response =
        chain.nextCall(request);

long duration =
        System.currentTimeMillis() - start;

log.info("LLM latency = {} ms", duration);

return response;
```

Now every AI request gets latency information automatically.

You could extend this to record:

```text
model
latency
tokens
errors
request type
tenant
```

This will connect nicely to your later **Observability** phase.

---

# 13. Before and after behavior

A custom Advisor can therefore operate on both sides of the call.

### Before LLM

```text
Request
  ↓
Validate
  ↓
Authorize
  ↓
Modify
  ↓
Enrich
  ↓
LLM
```

### After LLM

```text
LLM
 ↓
Inspect response
 ↓
Validate
 ↓
Log
 ↓
Transform
 ↓
Return
```

So the general pattern is:

```text
          BEFORE
             ↓
Request → Advisor → next()
                    ↓
                   LLM
                    ↓
              response
                    ↓
          AFTER processing
                    ↓
                 Client
```

---

# 14. Multiple custom Advisors

Now let's build your roadmap example.

Your roadmap gives the conceptual example:

```text
Request
 ↓
Security Advisor
 ↓
Logging Advisor
 ↓
RAG Advisor
 ↓
LLM
```



Let's make it concrete.

### Security Advisor

```text
Is user allowed?
```

### Logging Advisor

```text
Record request/response
```

### RAG Advisor

```text
Retrieve relevant documents
```

### LLM

```text
Generate answer
```

Final architecture:

```text
                    ┌──────────────────┐
                    │ SecurityAdvisor  │
                    └────────┬─────────┘
                             ↓
                    ┌──────────────────┐
                    │ LoggingAdvisor   │
                    └────────┬─────────┘
                             ↓
                    ┌──────────────────┐
                    │ RAGAdvisor      │
                    └────────┬─────────┘
                             ↓
                           LLM
```

---

# 15. Ordering matters

This is another **architect-level** concept.

Suppose you have:

```text
SecurityAdvisor
RAGAdvisor
```

You generally don't want:

```text
User
 ↓
RAG
 ↓
Retrieve confidential information
 ↓
Security check
```

because you've already performed the sensitive operation.

You'd normally want the authorization concern handled before the protected operation.

So:

```text
User
 ↓
Security
 ↓
RAG
 ↓
LLM
```

Advisor ordering can therefore affect correctness and security.

This is exactly the kind of thing you should think about when designing production AI systems.

---

# 16. Custom Advisor vs Service

A common question is:

> "Why not just put this logic in my service?"

You absolutely can.

For example:

```java
@Service
class AIService {

    public String ask(String question) {

        checkSecurity();
        addContext();
        logRequest();

        return chatClient
                .prompt()
                .user(question)
                .call()
                .content();
    }
}
```

That's perfectly reasonable for simple applications.

But once the same behavior applies to **many ChatClient interactions**, an Advisor becomes attractive.

### Service

Good for:

```text
Business-specific workflow
```

### Advisor

Good for:

```text
Cross-cutting AI interaction behavior
```

Examples:

```text
Logging
Security
Context enrichment
Request transformation
Response validation
Metrics
Tracing
Policy enforcement
```

---

# 17. Advisor vs Tool

These are very different.

### Advisor

Controls/intercepts the AI interaction:

```text
Request
 ↓
Advisor
 ↓
LLM
```

### Tool

Allows the LLM to invoke external functionality:

```text
User
 ↓
LLM
 ↓
"I need weather information"
 ↓
Weather Tool
 ↓
Weather API
 ↓
LLM
 ↓
Answer
```

So:

```text
Advisor → controls the AI pipeline

Tool → gives the AI an external capability
```

You'll study Tools later in Phase 12. 

---

# 18. Advisor vs Agent

Similarly:

```text
Advisor
```

is infrastructure around an interaction.

An:

```text
Agent
```

can involve an iterative reasoning/action loop:

```text
Goal
 ↓
LLM
 ↓
Tool
 ↓
Result
 ↓
LLM
 ↓
Tool
 ↓
Result
 ↓
Final Answer
```

That's why your roadmap deliberately puts Agents much later. 

---

# 19. A realistic architecture for you

Given your Spring Boot + React background, imagine you're building:

**Enterprise AI Assistant**

```text
                         React
                           │
                           ↓
                    Spring Boot API
                           │
                           ↓
                      ChatClient
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
    SecurityAdvisor   TenantAdvisor   Observability
          │                │                │
          └────────────────┼────────────────┘
                           ↓
                    RetrievalAdvisor
                           ↓
                     PostgreSQL
                      + pgvector
                           ↓
                          LLM
```

The key advantage is that the business service doesn't need to know every cross-cutting concern.

It can simply do:

```java
chatClient
    .prompt()
    .user(question)
    .call();
```

while the configured advisor chain handles the surrounding infrastructure.

---

# 20. The biggest takeaway

Don't think of a Custom Advisor as:

> "A special Spring AI class I need to implement."

Think of it as:

> **A reusable middleware component that participates in the lifecycle of an AI request/response.**

The mental model is:

```text
                 ┌─────────────────┐
                 │ Custom Advisor  │
                 │                 │
Request ────────→│ inspect         │
                 │ modify          │
                 │ validate        │
                 │ enrich          │
                 │ log             │
                 │                 │
                 │      next() ────┼────→ Next Advisor / LLM
                 │                 │
Response ←───────│ inspect         │
                 │ validate        │
                 │ transform       │
                 └─────────────────┘
```

And multiple Advisors compose into:

```text
Request
  ↓
Security
  ↓
Tenant
  ↓
Memory
  ↓
RAG
  ↓
Observability
  ↓
LLM
  ↓
Response
```

That is the **real architectural value** of Custom Advisors.

### Your hands-on exercise

For this topic, I'd recommend implementing **one simple `LoggingAdvisor` yourself** first:

```text
ChatClient
    ↓
LoggingAdvisor
    ↓
LLM
```

Have it log:

```text
→ AI request
→ model call started
← AI response
← execution time
```

Then implement a **SecurityAdvisor**:

```text
ChatClient
    ↓
SecurityAdvisor
    ↓
LLM
```

That second one will make the `chain.next...()` concept really click.

After that, you're ready for **Phase 11 — Chat Memory**, where you'll see how memory and advisors fit together in a real conversational application.

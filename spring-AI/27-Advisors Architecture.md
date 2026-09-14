# Advisor Architecture

### 1. What is an Advisor?

In Spring AI, an **Advisor** is a component that can **intercept and modify an AI interaction** before and/or after it reaches the LLM.

Think of it like a Spring `Filter` or an HTTP interceptor, but for an AI conversation.

Without an Advisor:

```text
Your Code
   ↓
ChatClient
   ↓
LLM
   ↓
Response
```

With Advisors:

```text
Your Code
   ↓
ChatClient
   ↓
Advisor 1
   ↓
Advisor 2
   ↓
Advisor 3
   ↓
LLM
   ↓
Advisor 3
   ↓
Advisor 2
   ↓
Advisor 1
   ↓
Response
```

The important idea is:

> **Advisors allow you to add reusable behavior around an LLM call without putting that logic directly into your business code.**

---

# 2. Why do we need Advisors?

Imagine your application has this:

```java
String response = chatClient
        .prompt()
        .user("What is our leave policy?")
        .call()
        .content();
```

Initially, this is simple.

But eventually you might want:

* conversation memory
* RAG retrieval
* logging
* retry handling
* security checks
* prompt modification
* adding system instructions
* monitoring
* token/cost tracking
* filtering requests/responses

You could put all of this into your service:

```java
public String ask(String question) {

    // Load conversation history

    // Search vector database

    // Build RAG context

    // Modify prompt

    // Log request

    // Call LLM

    // Log response

    // Save conversation

    // Return response
}
```

That quickly becomes messy.

Instead:

```text
ChatClient
   │
   ├── Memory Advisor
   ├── RAG Advisor
   ├── Logging Advisor
   └── Security Advisor
          │
          ↓
         LLM
```

Each concern becomes reusable and composable.

---

# 3. Advisor architecture

The architecture is essentially a **chain of responsibility**.

Imagine:

```text
                 ChatClient
                    │
                    ▼
            ┌────────────────┐
            │   Advisor A    │
            └────────────────┘
                    │
                    ▼
            ┌────────────────┐
            │   Advisor B    │
            └────────────────┘
                    │
                    ▼
            ┌────────────────┐
            │   Advisor C    │
            └────────────────┘
                    │
                    ▼
                   LLM
                    │
                    ▼
            ┌────────────────┐
            │   Advisor C    │
            └────────────────┘
                    │
                    ▼
            ┌────────────────┐
            │   Advisor B    │
            └────────────────┘
                    │
                    ▼
            ┌────────────────┐
            │   Advisor A    │
            └────────────────┘
                    │
                    ▼
                Response
```

An Advisor can therefore act:

### Before the LLM call

For example:

```text
User question
      ↓
Advisor
      ↓
Add conversation history
      ↓
Add RAG context
      ↓
Modify prompt
      ↓
LLM
```

### After the LLM call

For example:

```text
LLM response
      ↓
Advisor
      ↓
Log response
      ↓
Validate response
      ↓
Return to application
```

Some Advisors can do both.

---

# 4. The two important concepts

When learning Advisor architecture, keep these two concepts separate:

### `Advisor`

Represents the reusable behavior/interceptor.

### `AdvisorChain`

Represents the sequence in which Advisors execute.

Conceptually:

```text
Advisor
   ↓
Advisor
   ↓
Advisor
   ↓
LLM
```

The chain controls how the request moves through the Advisors.

---

# 5. What does an Advisor receive?

Conceptually, an Advisor receives an **AI interaction context**.

Think of it as:

```text
Advised request
├── Messages
├── User prompt
├── System prompt
├── Metadata
└── Other request information
```

The Advisor can inspect or modify this information.

For example:

```text
Original:

User:
"What is the leave policy?"

        ↓

RAG Advisor adds:

System:
"Answer using the following context..."

Context:
"Employees are entitled to 30 days..."

User:
"What is the leave policy?"
```

The LLM now receives the modified interaction.

---

# 6. A simple custom Advisor

Let's build a conceptual Advisor.

Suppose you want to log every question.

Conceptually:

```java
public class LoggingAdvisor implements Advisor {

    @Override
    public ChatClientRequest before(
            ChatClientRequest request,
            AdvisorChain chain) {

        System.out.println("User request: "
                + request.prompt());

        return chain.nextCall(request);
    }
}
```

The exact method signatures can vary with your Spring AI version, but architecturally this is what is happening:

```text
LoggingAdvisor
      │
      │ inspect request
      ▼
  log request
      │
      │ continue
      ▼
 AdvisorChain
      │
      ▼
    Next Advisor
```

The critical operation is:

```text
chain.next(...)
```

It means:

> "I have finished my work. Continue processing the request."

---

# 7. What happens if an Advisor doesn't continue?

This is important.

Suppose:

```java
public ChatClientRequest before(...) {

    if (requestIsInvalid) {
        return error;
    }

    return chain.nextCall(request);
}
```

If the Advisor decides **not** to continue, the LLM call can be prevented.

Therefore Advisors can also be used for things such as:

```text
Security validation
       ↓
Is request allowed?
       │
   ┌───┴────┐
   │        │
  NO       YES
   │        │
Reject    Continue
            ↓
           LLM
```

This makes Advisors useful beyond just prompt manipulation.

---

# 8. Advisors can modify the request

Suppose the application sends:

```text
"What is our leave policy?"
```

An Advisor could transform the interaction into:

```text
System:
You are an HR assistant.

Context:
Employees receive 30 days of annual leave.

User:
What is our leave policy?
```

The application itself doesn't need to know how that happened.

The flow becomes:

```text
Application
     │
     │ question
     ▼
ChatClient
     │
     ▼
RAG Advisor
     │
     ├── Search VectorStore
     │
     ├── Retrieve Documents
     │
     └── Add Context
     │
     ▼
LLM
```

This is one of the biggest reasons Advisors are useful for **RAG**.

---

# 9. Multiple Advisors

This is where the architecture becomes really powerful.

Imagine:

```java
chatClient
    .prompt()
    .advisors(
        memoryAdvisor,
        ragAdvisor,
        loggingAdvisor
    )
    .user(question)
    .call();
```

Conceptually:

```text
                  Question
                     │
                     ▼
              Memory Advisor
                     │
                     ▼
                RAG Advisor
                     │
                     ▼
             Logging Advisor
                     │
                     ▼
                    LLM
```

Each Advisor handles one concern.

For example:

### Memory Advisor

```text
"What did I ask earlier?"
       ↓
Retrieve conversation history
       ↓
Add history to request
```

### RAG Advisor

```text
Question
   ↓
Vector search
   ↓
Relevant documents
   ↓
Add context
```

### Logging Advisor

```text
Request
   ↓
Log
   ↓
Continue
```

This follows a very familiar software architecture principle:

> **Separate cross-cutting concerns into reusable components.**

---

# 10. Advisor ordering is important

This is a very important part of the architecture.

Suppose you have:

```text
Advisor A
Advisor B
Advisor C
LLM
```

The execution is roughly:

```text
A before
   ↓
B before
   ↓
C before
   ↓
LLM
   ↓
C after
   ↓
B after
   ↓
A after
```

This is similar to nested function calls.

Think:

```java
A(() -> 
    B(() ->
        C(() ->
            LLM()
        )
    )
)
```

So Advisor ordering can affect the final prompt and response.

For example:

```text
Memory
   ↓
RAG
   ↓
Prompt modification
   ↓
LLM
```

can produce a different result from:

```text
RAG
   ↓
Memory
   ↓
Prompt modification
   ↓
LLM
```

Therefore, when using multiple Advisors, you need to understand their order.

---

# 11. Advisor vs Service class

This distinction is important.

You might ask:

> Why not just create a `RagService`?

You absolutely can.

For example:

```java
@Service
public class RagService {

    public String answer(String question) {
        // retrieve documents
        // build prompt
        // call LLM
    }
}
```

But now your application code is coupled to the RAG workflow.

With an Advisor:

```text
Application
    ↓
ChatClient
    ↓
RAG Advisor
    ↓
LLM
```

The RAG behavior becomes something that can be attached to different AI interactions.

For example:

```text
ChatClient A
   └── RAG Advisor

ChatClient B
   └── Memory Advisor

ChatClient C
   ├── Memory Advisor
   ├── RAG Advisor
   └── Logging Advisor
```

This is **composition** rather than putting everything into one service.

---

# 12. Advisor vs Spring AOP

You may notice a similarity with Spring AOP.

Spring AOP:

```text
Controller
    ↓
Aspect
    ↓
Service
    ↓
Database
```

Spring AI Advisors:

```text
Application
    ↓
Advisor
    ↓
Advisor
    ↓
LLM
```

Both provide interception and cross-cutting behavior.

But they operate at different levels.

### Spring AOP

Generally concerned with application method execution:

```text
method()
```

### Spring AI Advisor

Specifically understands the AI interaction:

```text
Prompt
Messages
ChatClient
LLM call
Response
```

That's why an Advisor can naturally manipulate things like prompts, messages, retrieved context, and AI responses.

---

# 13. The RAG connection

You've already learned:

```text
Question
   ↓
Embedding
   ↓
Vector Store
   ↓
Relevant Documents
   ↓
Context
   ↓
Prompt
   ↓
LLM
```

Advisor architecture allows Spring AI to package much of this workflow into reusable components.

Instead of manually doing:

```java
List<Document> documents =
        vectorStore.similaritySearch(...);

String context = ...;

String prompt = ...;

chatClient
        .prompt()
        .user(prompt)
        .call();
```

you can conceptually have:

```text
Question
   ↓
RAG Advisor
   ├── Retrieve documents
   ├── Build context
   └── Modify request
   ↓
LLM
```

So you can think of Advisors as the **extension/interception mechanism that Spring AI uses to build reusable AI workflows around ChatClient calls**.

---

# 14. A complete example architecture

Imagine your HR chatbot.

User asks:

```text
"How many vacation days do I get?"
```

Your application:

```java
String answer = chatClient
        .prompt()
        .user("How many vacation days do I get?")
        .call()
        .content();
```

Now add Advisors:

```text
                     User Question
                           │
                           ▼
                    ┌─────────────┐
                    │ ChatClient  │
                    └──────┬──────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │ Memory Advisor  │
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │  RAG Advisor    │
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │ Logging Advisor │
                  └────────┬────────┘
                           │
                           ▼
                         LLM
                           │
                           ▼
                  ┌─────────────────┐
                  │ Logging Advisor │
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │ Memory Advisor  │
                  └────────┬────────┘
                           │
                           ▼
                       Response
```

The application doesn't need to manually orchestrate all of this.

---

# 15. The mental model I recommend

Since you already know Spring Boot, remember Advisors using this analogy:

```text
Spring Web:

Request
   ↓
Filter
   ↓
Filter
   ↓
Controller
   ↓
Response
```

Spring AI:

```text
AI Request
   ↓
Advisor
   ↓
Advisor
   ↓
LLM
   ↓
Advisor
   ↓
Response
```

So:

> **Advisor = interceptor/filter for an AI interaction.**

And:

> **Advisor Chain = ordered pipeline of those interceptors.**

---

# 16. What you should know after this topic

For **Advisor Architecture**, make sure these are clear:

```text
✓ What is an Advisor?
✓ Why Advisors exist
✓ Advisor chain
✓ Before/around/after processing
✓ Request modification
✓ Response processing
✓ Multiple Advisors
✓ Advisor ordering
✓ Chain continuation
✓ Custom Advisors
✓ Cross-cutting concerns
✓ Relationship with RAG
✓ Advisor vs service class
✓ Advisor vs Spring AOP
```

The most important picture to remember is:

```text
                 ┌─────────────────┐
                 │    ChatClient   │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │    Advisor 1    │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │    Advisor 2    │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │    Advisor 3    │
                 └────────┬────────┘
                          │
                          ▼
                         LLM
                          │
                          ▼
                     Response
```

**One-line definition for your notes:**

> **Spring AI Advisor is a reusable interceptor that can inspect, modify, enrich, validate, or process an AI request/response as it passes through an ordered Advisor chain around an LLM call.**

The next useful topic is **built-in Advisors**, where this architecture becomes concrete with things like **Chat Memory, RAG, and Question Answering Advisors**.

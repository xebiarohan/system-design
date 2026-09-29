# Phase 2 — Chat Models

## 8. Conversation History — 2–4 hours

Conversation history is one of the most important concepts to understand when building a chat application with Spring AI.

The key idea is:

> **LLMs are generally stateless. Your application is responsible for maintaining the conversation context.**

---

## 1. The problem: LLMs are stateless

Suppose the user says:

```text
User:
My name is Rohan.
```

You send this to the LLM:

```text
User → LLM

"My name is Rohan."
```

The LLM responds:

```text
Assistant:
Nice to meet you, Rohan!
```

Now the user asks:

```text
What is my name?
```

If you send **only this new message**:

```text
User → LLM

"What is my name?"
```

The LLM may not know.

Why?

Because the LLM does not automatically remember the previous API request.

Conceptually:

```text
Request 1
User: My name is Rohan.
        ↓
       LLM
        ↓
Response

        ❌ Nothing automatically remembered

Request 2
User: What is my name?
        ↓
       LLM
```

Each request can be treated independently.

---

# 2. How conversation history solves this

Your application keeps the previous messages.

Instead of sending:

```text
User:
What is my name?
```

your application sends:

```text
System:
You are a helpful assistant.

User:
My name is Rohan.

Assistant:
Nice to meet you, Rohan!

User:
What is my name?
```

Now the LLM has the context it needs.

The architecture becomes:

```text
                 ┌──────────────────┐
                 │ Conversation DB  │
                 └────────┬─────────┘
                          │
                          │ history
                          ↓
User → Spring Boot → Spring AI → LLM
                          ↑
                          │
                     new message
```

The important point is:

```text
Conversation history
        ↓
added to current prompt
        ↓
LLM
```

The LLM isn't necessarily "remembering" anything.

**Your application is providing the memory/context again.**

---

# 3. Stateless vs Stateful thinking

This distinction is extremely important.

### Stateless LLM interaction

```text
Request 1
   ↓
LLM
   ↓
Response

Request 2
   ↓
LLM
   ↓
Response
```

The requests are independent.

### Stateful application

Your application creates the illusion of a continuous conversation:

```text
User
 ↓
Spring Boot
 ↓
Load conversation history
 ↓
Add new message
 ↓
Spring AI
 ↓
LLM
 ↓
Response
 ↓
Save messages
```

So:

```text
LLM = stateless

Application = manages state
```

This is a very useful mental model.

---

# 4. What exactly is conversation history?

Conversation history is simply a collection of messages.

For example:

```text
[
    SystemMessage("You are a helpful assistant"),

    UserMessage("My name is Rohan"),

    AssistantMessage("Nice to meet you, Rohan"),

    UserMessage("What is my name?")
]
```

Spring AI represents these using its message abstraction.

Conceptually:

```text
Message
   │
   ├── SystemMessage
   ├── UserMessage
   └── AssistantMessage
```

The conversation is therefore not just a string.

It is a sequence of messages with different roles.

---

# 5. Why message roles matter

Consider:

```text
System:
You are a Java expert.

User:
Explain dependency injection.

Assistant:
Dependency injection is...
```

These messages have different meanings.

### System message

Defines behavior/instructions:

```text
You are a Java expert.
Answer using Java examples.
```

### User message

Represents the user's request:

```text
Explain dependency injection.
```

### Assistant message

Represents the model's previous response:

```text
Dependency injection is a design pattern...
```

The complete conversation can therefore be viewed as:

```text
┌───────────────┐
│ System        │
├───────────────┤
│ User          │
├───────────────┤
│ Assistant     │
├───────────────┤
│ User          │
├───────────────┤
│ Assistant     │
└───────────────┘
```

---

# 6. Conversation ID

In a real application, you need to know:

> Which conversation does this message belong to?

For example:

```text
conversationId = "abc123"
```

User sends:

```text
POST /chat/abc123

"My name is Rohan."
```

You save:

```text
conversationId: abc123
role: USER
content: My name is Rohan.
```

The AI responds:

```text
Nice to meet you, Rohan!
```

You save:

```text
conversationId: abc123
role: ASSISTANT
content: Nice to meet you, Rohan!
```

Later:

```text
POST /chat/abc123

"What is my name?"
```

Your backend retrieves:

```text
conversationId = abc123

History:
------------------------------
USER:
My name is Rohan.

ASSISTANT:
Nice to meet you, Rohan!
------------------------------
```

Then adds:

```text
USER:
What is my name?
```

and sends the conversation to the LLM.

---

# 7. Multiple users and multiple conversations

This becomes especially important in a real application.

Imagine:

```text
User A
Conversation 1
conversationId = c001

User A
Conversation 2
conversationId = c002

User B
Conversation 1
conversationId = c003
```

You must keep these conversations separate.

For example:

```text
conversationId = c001

User:
My name is Rohan.
```

should **never** accidentally appear in:

```text
conversationId = c003
```

A typical database design could be:

```text
conversation
--------------------------------
id
user_id
created_at
title
```

and:

```text
message
--------------------------------
id
conversation_id
role
content
created_at
```

Relationship:

```text
User
 │
 ├── Conversation
 │      │
 │      ├── Message
 │      ├── Message
 │      └── Message
 │
 └── Conversation
        │
        ├── Message
        └── Message
```

---

# 8. Simple Spring AI approach

At the simplest level, you can maintain the messages yourself.

Conceptually:

```java
List<Message> messages = new ArrayList<>();

messages.add(new SystemMessage("You are a helpful assistant"));
messages.add(new UserMessage("My name is Rohan"));
messages.add(new AssistantMessage("Nice to meet you, Rohan"));
messages.add(new UserMessage("What is my name?"));
```

Then:

```java
Prompt prompt = new Prompt(messages);

ChatResponse response = chatModel.call(prompt);
```

The important thing here isn't memorizing the exact API.

Understand the architecture:

```text
Conversation history
        ↓
List<Message>
        ↓
Prompt
        ↓
ChatModel
        ↓
LLM
```

---

# 9. Using ChatClient

With `ChatClient`, you can construct the request more naturally.

Conceptually:

```java
chatClient
    .prompt()
    .system("You are a helpful assistant")
    .user("My name is Rohan")
    .call();
```

For the next request, if you want the LLM to understand the previous interaction, you need to provide the relevant history.

Conceptually:

```java
chatClient
    .prompt()
    .system("You are a helpful assistant")
    .messages(history)
    .user("What is my name?")
    .call();
```

The important concept is:

```text
ChatClient
   ↓
Prompt
   ↓
Messages
   ↓
LLM
```

---

# 10. Where should conversation history be stored?

There are several possibilities.

### Option 1 — In memory

For example:

```java
Map<String, List<Message>>
```

Architecture:

```text
Spring Boot
    ↓
Memory
    ↓
Conversation history
```

Good for:

* Learning
* Small prototypes
* Local development

Bad for:

* Production
* Multiple application instances
* Application restarts

If your application restarts:

```text
Application
    ↓
💥 Restart
    ↓
History lost
```

---

# 11. Database storage

For production applications, conversation history is commonly persisted.

For example:

```text
PostgreSQL

conversation
-------------------------
id
user_id
created_at

message
-------------------------
id
conversation_id
role
content
created_at
```

Then:

```text
User
 ↓
Spring Boot
 ↓
PostgreSQL
 ↓
Conversation History
 ↓
Spring AI
 ↓
LLM
```

Now history survives application restarts.

---

# 12. Redis

Redis is another option.

For example:

```text
conversation:c001
        ↓
[message1, message2, message3]
```

Advantages:

* Very fast
* Good for temporary/session-oriented state
* Useful for distributed applications

But Redis isn't automatically the correct choice for every application.

For long-term conversation persistence, a relational database can often be simpler.

---

# 13. Spring AI Chat Memory

Spring AI provides abstractions specifically for conversation memory.

The idea is:

```text
ChatClient
     ↓
Memory / Advisor
     ↓
Conversation history
     ↓
LLM
```

Instead of manually doing:

```text
Load messages
      ↓
Add messages
      ↓
Build prompt
      ↓
Call LLM
      ↓
Save messages
```

Spring AI can help manage this through its memory-related abstractions/advisors.

This becomes especially important in the next roadmap phase:

```text
Phase 11 — Chat Memory
```

For **this topic**, focus on understanding the underlying concept first.

Don't jump immediately into memorizing the Spring AI memory APIs.

---

# 14. Conversation history ≠ long-term memory

This distinction is VERY important.

Suppose the conversation is:

```text
User:
My name is Rohan.

Assistant:
Nice to meet you.

User:
I work with Java.

Assistant:
Great!

User:
What language do I work with?
```

Conversation history gives the model:

```text
My name is Rohan.
I work with Java.
```

That's conversation context.

But imagine six months later the user starts a completely new conversation:

```text
User:
What programming language do I usually work with?
```

Should the AI know?

That's a different problem.

You are now talking about:

```text
Long-term memory
```

rather than simply:

```text
Conversation history
```

Think:

```text
Conversation History
        │
        │
        ├── Current conversation
        │
        └── Usually limited in scope


Long-Term Memory
        │
        ├── User preferences
        ├── Important facts
        ├── Previous interactions
        └── Persistent knowledge
```

Your roadmap handles this more deeply in:

```text
Phase 11 — Chat Memory
```

---

# 15. The context window problem

This is one of the most important limitations.

Suppose a conversation becomes very long:

```text
Message 1
Message 2
Message 3
...
Message 1000
```

You might think:

```text
Send all 1000 messages to the LLM
```

But that's problematic.

Every LLM has a context window.

Conceptually:

```text
Context Window
┌───────────────────────────────┐
│ System prompt                 │
│ Conversation history          │
│ Current user message          │
│ Retrieved documents           │
│ Tool results                  │
└───────────────────────────────┘
```

Everything must fit within the available context.

---

# 16. Context window and cost

Long history also increases cost.

For example:

```text
Request 1
10 tokens history
+
10 new tokens
```

Cheap.

But:

```text
Request 100
10,000 tokens history
+
50 new tokens
```

Now you're repeatedly sending thousands of tokens.

So conversation history affects:

* Context window
* Latency
* Token usage
* Cost
* Model performance

This is why production chat systems don't simply keep adding messages forever.

---

# 17. Strategies for long conversations

Eventually you need strategies such as:

### Strategy 1 — Keep recent messages

For example:

```text
Message 1
Message 2
Message 3
...
Message 95
Message 96
Message 97
Message 98
Message 99
Message 100
```

Only send:

```text
Message 91 → 100
```

This is often called a sliding window approach.

---

### Strategy 2 — Summarization

Instead of keeping:

```text
Message 1
Message 2
Message 3
...
Message 50
```

create:

```text
Summary:

User is Rohan.
He is a Java developer.
He is building a Spring Boot application.
He prefers PostgreSQL.
The current problem is authentication.
```

Then:

```text
Summary
   +
Recent messages
   ↓
LLM
```

This dramatically reduces the amount of context.

---

### Strategy 3 — Retrieval-based memory

Store important information separately.

For example:

```text
User facts
-------------------------
Name = Rohan
Profession = Java Developer
Preference = PostgreSQL
Project = Spring AI chatbot
```

Then retrieve relevant memories when needed.

This starts moving toward:

```text
Memory + RAG
```

which you'll study later.

---

# 18. Conversation history vs RAG

These two concepts are easy to confuse.

### Conversation history

Answers:

> What have we talked about in this conversation?

```text
Previous messages
        ↓
Current prompt
        ↓
LLM
```

### RAG

Answers:

> What information exists in my external knowledge base that is relevant to this question?

```text
Question
   ↓
Vector Search
   ↓
Relevant documents
   ↓
Prompt
   ↓
LLM
```

So:

```text
Conversation History
        ↓
Previous conversation

RAG
        ↓
External knowledge
```

A production AI application can use both:

```text
                 ┌── Conversation History
                 │
User Question ───┼── RAG Documents
                 │
                 └── System Instructions
                          ↓
                         LLM
```

---

# 19. Complete chat request flow

A realistic Spring AI chat application could look like this:

```text
                 User
                  │
                  ↓
          React Chat UI
                  │
                  ↓
          Spring Boot API
                  │
                  ↓
       conversationId = c001
                  │
                  ↓
       Load conversation history
                  │
                  ↓
        ┌───────────────────┐
        │ System message    │
        │ Previous messages │
        │ New user message  │
        └─────────┬─────────┘
                  │
                  ↓
             ChatClient
                  │
                  ↓
                 LLM
                  │
                  ↓
           AI response
                  │
          ┌───────┴────────┐
          ↓                ↓
     Return to UI     Save response
```

This is the architecture you should have in your head.

---

# 20. A simple example

Imagine the user has a chat:

### Message 1

```text
User:
I'm building an e-commerce application using Spring Boot.
```

History:

```text
USER:
I'm building an e-commerce application using Spring Boot.
```

### Message 2

```text
User:
I need authentication.
```

The application sends:

```text
SYSTEM:
You are a Spring Boot expert.

USER:
I'm building an e-commerce application using Spring Boot.

USER:
I need authentication.
```

The AI responds:

```text
ASSISTANT:
You can use Spring Security...
```

Now the user asks:

```text
What database should I use?
```

The application sends something like:

```text
SYSTEM:
You are a Spring Boot expert.

USER:
I'm building an e-commerce application using Spring Boot.

USER:
I need authentication.

ASSISTANT:
You can use Spring Security...

USER:
What database should I use?
```

The LLM can now understand:

```text
"What database should I use?"
```

in the context of:

```text
Spring Boot
+
E-commerce
+
Authentication
```

Without history, the question is ambiguous.

---

# 21. The most important mental model

Don't think:

```text
LLM remembers the conversation.
```

Think:

```text
                    ┌───────────────┐
                    │ Conversation  │
                    │    Storage    │
                    └───────┬───────┘
                            │
                         History
                            │
                            ↓
User → Spring Boot → Spring AI → LLM
                            │
                            ↓
                         Response
                            │
                            ↓
                     Save to storage
```

The application creates the **illusion of memory** by continuously providing relevant previous context.

---

# 22. What you should know before moving on

After completing this topic, you should be able to explain all of these:

### Statelessness

```text
LLM requests are generally independent.
```

### Conversation history

```text
Previous messages
        ↓
Current request
        ↓
LLM
```

### Message roles

```text
System
User
Assistant
```

### Conversation ID

```text
User
 ↓
Conversation ID
 ↓
Messages
```

### Persistence

```text
In-memory
   vs
Database
   vs
Redis
```

### Context window

```text
More history
     ↓
More tokens
     ↓
Higher cost + possible context limit
```

### History management

```text
Recent messages
+
Summary
+
Relevant memory
```

### History vs RAG

```text
History → previous conversation

RAG → external knowledge
```

---

# 🧪 Hands-on Exercise

Build a very small chat API.

### Step 1

Create:

```text
POST /chat/{conversationId}
```

Request:

```json
{
  "message": "My name is Rohan"
}
```

### Step 2

Store messages in memory:

```text
Map<ConversationId, List<Message>>
```

### Step 3

For every request:

```text
conversationId
      ↓
Load history
      ↓
Add user message
      ↓
Call ChatClient
      ↓
Add assistant response
      ↓
Return response
```

### Step 4

Test this sequence:

```text
User:
My name is Rohan.

AI:
Nice to meet you, Rohan.

User:
What is my name?

AI:
Your name is Rohan.
```

### Step 5

Test a second conversation:

```text
conversationId = 100

User:
My name is Rohan.
```

and:

```text
conversationId = 200

User:
What is my name?
```

The second conversation **should not know** Rohan's name.

That exercise will make the concept click much faster than just reading about ChatMemory.

---

# 🎯 Topic 8 Goal

By the end of this topic, your mental model should be:

```text
             Conversation ID
                    │
                    ↓
            Conversation History
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
    Previous Messages     New Message
          │                   │
          └─────────┬─────────┘
                    ↓
                ChatClient
                    ↓
                   LLM
                    ↓
                AI Response
                    │
                    ↓
          Save to Conversation
```

And the single most important sentence to remember is:

> **The LLM doesn't magically remember previous requests; the application maintains conversation state and provides the relevant history to the model.**

### What comes next

You already have this roadmap split into two related topics:

```text
8. Conversation history
        ↓
Understand the concept

Phase 11 — Chat Memory
        ↓
Learn how Spring AI manages memory,
persistence, token limits, summarization,
and production-scale conversation context
```

So **don't spend too much time trying to master every ChatMemory API right now**. For Topic 8, focus on the architecture and build the small in-memory chat exercise. That will make Phase 11 much easier.

# Chat Memory

## The big picture

Suppose the user says:

```text
User: My name is Rohan.
AI: Nice to meet you, Rohan!
```

Then a few minutes later:

```text
User: What is my name?
```

If you make the second request independently:

```text
User
  ↓
Spring AI
  ↓
LLM
```

the LLM normally has **no idea** what happened in the previous request.

You need:

```text
                    ┌──────────────────┐
                    │ Conversation     │
                    │ History          │
                    └────────┬─────────┘
                             │
User message                 │
     │                       │
     └──────────┬────────────┘
                ↓
          Spring AI
                ↓
               LLM
                ↓
             Response
                │
                ↓
        Store conversation
```

So the core problem is:

```text
How do I maintain conversational context across multiple requests?
```

That's what this phase teaches.

---

# 35. Chat Memory

**⏱️ ~2 hours**

This is the foundation of the entire phase.

## 1. Stateless chat

First understand what happens without memory.

Imagine:

### Request 1

```text
POST /chat

{
    "message": "My name is Rohan"
}
```

Spring AI sends:

```text
User:
My name is Rohan
```

LLM responds:

```text
Nice to meet you, Rohan!
```

Now request 2:

```text
POST /chat

{
    "message": "What is my name?"
}
```

If you send only:

```text
User:
What is my name?
```

the model doesn't necessarily know that the previous conversation existed.

---

# 2. Adding conversation history

Instead, your application can send:

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

Now the LLM has the required context.

So conceptually:

```text
Conversation

Message 1
   ↓
Message 2
   ↓
Message 3
   ↓
Message 4
   ↓
...
```

Spring AI's memory facilities help manage this history.

---

# 3. Important distinction: memory vs context

This distinction is **very important for interviews and architecture**.

### Memory

Where your application keeps the conversation history.

For example:

```text
Redis
PostgreSQL
MongoDB
In-memory Map
```

### Context

The information you actually provide to the LLM for the current request.

For example:

```text
Previous messages
+
Current user message
```

So:

```text
Persistent storage
       ↓
Conversation history
       ↓
Select relevant messages
       ↓
Prompt/context
       ↓
LLM
```

The LLM only sees what you put into its request.

---

# 4. Spring AI's role

Think of Spring AI as helping you manage this pipeline:

```text
User
 ↓
ChatClient
 ↓
Memory / Advisor
 ↓
Build prompt with history
 ↓
LLM
 ↓
Response
 ↓
Update memory
```

This connects directly with the **Advisor** phase you just completed.

You already learned:

```text
Request
   ↓
Advisor
   ↓
Advisor
   ↓
ChatClient
   ↓
LLM
```

Memory can participate in that pipeline.

So conceptually:

```text
User request
     ↓
Conversation Memory Advisor
     ↓
Retrieve previous messages
     ↓
ChatClient
     ↓
LLM
     ↓
Response
     ↓
Update memory
```

That's why **Advisors → Memory** is a sensible progression in your roadmap.

---

# 36. Conversation ID

**⏱️ ~1 hour**

This is a deceptively important topic.

Imagine your application has:

```text
User A
User B
```

User A says:

```text
My name is Rohan.
```

User B says:

```text
My name is Amit.
```

You obviously cannot put both conversations into one memory.

You need to identify conversations.

That's where a **conversation ID** comes in.

---

## Example

Suppose:

```text
conversationId = "abc123"
```

User sends:

```text
POST /chat

{
    "conversationId": "abc123",
    "message": "My name is Rohan"
}
```

You store:

```text
abc123
 ├── User: My name is Rohan
 └── Assistant: Nice to meet you, Rohan
```

Next request:

```text
{
    "conversationId": "abc123",
    "message": "What is my name?"
}
```

Your application retrieves:

```text
abc123
```

and obtains:

```text
User: My name is Rohan
Assistant: Nice to meet you, Rohan
```

Then sends the relevant history to the LLM.

---

# Why Conversation ID matters

Without an identifier:

```text
All users
   ↓
One giant conversation
```

Obviously bad.

With IDs:

```text
conversation-001
      ↓
User A's conversation

conversation-002
      ↓
User B's conversation

conversation-003
      ↓
User A's second conversation
```

This also gives you the concept of **multiple conversations per user**.

For example:

```text
User: Rohan

Conversations
├── Java discussion
├── Spring AI discussion
└── Travel discussion
```

Each can have a different conversation ID.

---

# Conversation ID ≠ User ID

This is another useful architectural distinction.

You might have:

```text
userId = 42
```

and:

```text
conversationId = abc123
```

Relationship:

```text
User 42
 ├── Conversation abc123
 ├── Conversation xyz456
 └── Conversation pqr789
```

So typically:

```text
User
  │
  └── 1:N
       │
       ├── Conversation
       ├── Conversation
       └── Conversation
```

This becomes important when you eventually build a real ChatGPT-style application.

---

# 37. Persistent Memory

**⏱️ ~2–3 hours**

Now we get to the interesting part.

A simple implementation might store memory in:

```java
Map<String, List<Message>>
```

For example:

```text
Map
│
├── abc123 → messages
├── xyz456 → messages
└── pqr789 → messages
```

This works for a demo.

But what happens when your Spring Boot application restarts?

💥 The memory disappears.

That's **in-memory conversation history**.

---

# Persistent memory

Instead, store the conversation somewhere durable.

For example:

```text
                    Spring Boot
                         │
                         ↓
                  Chat Memory
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
          PostgreSQL              Redis
```

Now:

```text
Application restart
        ↓
Memory still exists
```

---

# Example database model

You could conceptually have:

### conversation

```text
conversation
-------------------------
id
user_id
created_at
updated_at
title
```

### messages

```text
message
-------------------------
id
conversation_id
role
content
created_at
```

For example:

```text
conversation
--------------------------------
id       user_id
abc123   42
```

Messages:

```text
id   conversation_id   role       content
------------------------------------------------
1    abc123            user       My name is Rohan
2    abc123            assistant  Nice to meet you
3    abc123            user       What is my name?
```

Then:

```text
conversation_id
       ↓
retrieve messages
       ↓
construct context
       ↓
LLM
```

---

# Why persistence matters

Consider a production system:

```text
React
  ↓
Load Balancer
  ↓
Spring Boot instance 1
Spring Boot instance 2
Spring Boot instance 3
```

Suppose memory is stored only inside JVM memory:

```text
Instance 1
   ↓
Memory
```

A subsequent request could hit:

```text
Instance 2
```

and suddenly:

```text
Where did my conversation go?
```

😄

Persistent/shared memory solves this:

```text
             ┌───────────────┐
             │   PostgreSQL  │
             │ / Redis       │
             └───────┬───────┘
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
     App 1         App 2        App 3
```

Now all instances can access the conversation.

This is particularly relevant to your **system-design / Architect path**.

---

# Persistent memory does NOT mean "send everything forever"

This leads directly to topic 38.

---

# 38. Token Limits & Summarization

**⏱️ ~1–2 hours**

This is probably the most architecturally important topic in this phase.

Your first conversation might contain:

```text
10 messages
```

No problem.

Then:

```text
100 messages
```

Still potentially okay.

Then:

```text
1,000 messages
```

Now you have a problem.

Your roadmap describes exactly this situation: sending the entire history to the LLM eventually runs into **context-window and cost problems**. 

---

# Why?

Every time you call the LLM, you're paying for input tokens too.

Imagine:

```text
Request 1
10 tokens
```

Then:

```text
Request 2
previous 10
+ new 10
= 20
```

Then:

```text
Request 3
previous 20
+ new 10
= 30
```

Eventually:

```text
100,000 tokens
```

Now you're dealing with:

### 1. Context window

The model has a maximum amount of context it can process.

### 2. Cost

More input tokens → higher cost for many models.

### 3. Latency

More input → potentially more processing.

### 4. Irrelevant history

Old messages may no longer be useful.

---

# Strategy 1 — Keep only recent messages

For example:

```text
Conversation:

Message 1
Message 2
Message 3
...
Message 98
Message 99
Message 100
```

Instead of sending everything:

```text
Send:

Message 91
Message 92
...
Message 100
```

This is essentially a **sliding window** approach.

```text
┌─────────────────────────────┐
│ Recent conversation         │
│                             │
│ M91 M92 M93 ... M100        │
└─────────────────────────────┘
```

Simple and cheap.

But you lose older information.

---

# Strategy 2 — Summarization

Suppose the conversation is:

```text
M1
M2
M3
...
M80
```

Instead of keeping all of it:

```text
M1-M70
   ↓
Summary
```

For example:

```text
Summary:

The user is building a Spring Boot application.
They use PostgreSQL.
They prefer Java.
They are currently implementing authentication.
```

Then:

```text
Summary
   +
M71
M72
M73
...
M80
   ↓
LLM
```

This dramatically reduces context size.

---

# Strategy 3 — Hybrid memory

In a real application, you might combine:

```text
Long-term summary
        +
Recent messages
        +
Relevant retrieved information
```

For example:

```text
                Conversation
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Summary    Recent      Relevant
                  messages     history
          │          │           │
          └──────────┼───────────┘
                     ↓
                  Prompt
                     ↓
                    LLM
```

This is much closer to how you should think about production conversational systems.

---

# Chat Memory vs RAG Memory

This distinction is **very important**, especially because you've already studied RAG.

They solve different problems.

## Chat Memory

Answers:

> "What happened earlier in this conversation?"

Example:

```text
User:
My name is Rohan.

...

User:
What is my name?
```

Memory provides:

```text
My name is Rohan.
```

---

## RAG

Answers:

> "What information do I need to retrieve from my knowledge base?"

Example:

```text
User:
What is our company's leave policy?
```

RAG retrieves:

```text
HR policy PDF
       ↓
Relevant chunks
       ↓
LLM
```

---

# Combine them

This is where Spring AI becomes really interesting.

You can have:

```text
                    User question
                         │
                         ↓
              ┌─────────────────────┐
              │ Conversation Memory │
              └──────────┬──────────┘
                         │
                         ↓
                 Previous context
                         │
                         ↓
              ┌─────────────────────┐
              │       RAG           │
              └──────────┬──────────┘
                         │
                         ↓
                  Relevant documents
                         │
                         ↓
                       LLM
```

For example:

```text
User:
What did we discuss yesterday about the leave policy?
```

Memory can provide:

```text
Previous conversation
```

RAG can provide:

```text
Current HR policy document
```

The LLM combines both.

---

# Your Phase 11 mental model

I'd remember the entire phase like this:

```text
                    USER
                     │
                     ↓
              conversationId
                     │
                     ↓
              Retrieve Memory
                     │
                     ↓
       ┌─────────────┴─────────────┐
       │                           │
       ↓                           ↓
 Conversation                  Current
   History                     Message
       │                           │
       └─────────────┬─────────────┘
                     ↓
                Build Context
                     │
                     ↓
                    LLM
                     │
                     ↓
                 Response
                     │
                     ↓
               Store Memory
                     │
                     ↓
             Persistent Store
```

And when history becomes huge:

```text
Huge conversation
       ↓
 ┌─────┴───────────┐
 ↓                 ↓
Recent          Summary
messages
 └──────┬──────────┘
        ↓
     Context
        ↓
       LLM
```

---

# What you should actually code

For your hands-on learning, I would make your existing Chat application evolve through **four iterations**.

### Step 1 — Stateless

```text
POST /chat
message
   ↓
LLM
```

No memory.

### Step 2 — Conversation ID

```text
POST /chat

{
  "conversationId": "abc123",
  "message": "My name is Rohan"
}
```

Now multiple conversations are possible.

### Step 3 — Persistent memory

Store:

```text
conversationId
+
messages
```

in PostgreSQL.

Then verify:

```text
Chat
 ↓
Restart Spring Boot
 ↓
Continue same conversation
 ↓
Memory still works
```

### Step 4 — Context management

Create a long conversation and implement something like:

```text
Recent N messages
        +
Conversation summary
```

This gives you hands-on experience with **all four topics in Phase 11**.

---

# The four topics at a glance

| Topic                                | Core question                                                   |
| ------------------------------------ | --------------------------------------------------------------- |
| **35. Chat Memory**                  | How does an application maintain conversational context?        |
| **36. Conversation ID**              | How do we separate different conversations?                     |
| **37. Persistent Memory**            | How do we preserve conversations across restarts/instances?     |
| **38. Token Limits & Summarization** | How do we prevent conversation history from becoming too large? |

And the final project from your roadmap is exactly the right exercise: **add persistent conversation history to your existing Chat application**. 

### One interview-level takeaway

If I asked you:

> **"Does ChatGPT remember my previous conversation because the LLM itself has memory?"**

A strong answer would be:

> **Not necessarily. The application maintains conversation state outside the model and supplies the relevant history as context when making subsequent model requests. That history can be stored in memory or a persistent store, identified by a conversation/session ID, and managed using techniques such as recent-message windows and summarization to control token usage and context size.**

That's the core of Phase 11.

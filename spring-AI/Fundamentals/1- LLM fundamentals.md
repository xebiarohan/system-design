# Phase 0.1 — LLM Fundamentals

Your first mental model should be:

```text
Your Spring Boot Application
          ↓
       Prompt
          ↓
      LLM API
          ↓
       Tokens
          ↓
    Model processing
          ↓
 Generated tokens
          ↓
      Response
```

An LLM is essentially a model that, given a sequence of tokens, predicts what tokens should come next.

That sounds simple, but there are several important concepts hidden inside that sentence.

---

# 1. What is an LLM?

**LLM = Large Language Model.**

Examples include models from OpenAI, Anthropic, Google, Meta, Mistral, etc.

At a high level, an LLM does this:

```text
Input
  ↓
LLM
  ↓
Predicted output
```

For example:

```text
Input:
"Explain dependency injection in Spring"

             ↓

            LLM

             ↓

Output:
"Dependency injection is a design pattern..."
```

But internally, the model doesn't really see:

> "Explain dependency injection in Spring"

as human-readable words.

It works with **tokens represented numerically**.

We'll get to tokens shortly.

---

# 2. How does an LLM actually generate text?

This is probably the most important fundamental concept.

Suppose you send:

```text
Spring Boot is
```

The model considers possible next tokens:

```text
Spring Boot is

  a
  an
  used
  designed
  ...
```

It assigns probabilities to possible next tokens.

For example, conceptually:

```text
"a"          → 0.35
"used"       → 0.25
"designed"   → 0.15
"an"         → 0.05
...
```

The model chooses a token according to its generation strategy.

Then:

```text
Spring Boot is a
```

The model predicts the next token again:

```text
Spring Boot is a

  framework
  technology
  Java
  ...
```

Then:

```text
Spring Boot is a framework
```

Again:

```text
Spring Boot is a framework

  for
  used
  that
  designed
  ...
```

This continues until the model decides to stop.

So conceptually:

```text
Prompt
  ↓
Predict next token
  ↓
Add token
  ↓
Predict next token
  ↓
Add token
  ↓
Predict next token
  ↓
...
  ↓
Final response
```

### Important

An LLM isn't simply retrieving a paragraph from a database.

It is **generating a sequence of tokens based on learned patterns and the supplied context**.

This becomes extremely important later when you learn:

* hallucinations
* temperature
* token limits
* RAG
* tool calling
* agents

---

# 3. Tokens

Tokens are one of the most important concepts you need to understand.

An LLM doesn't necessarily process one word at a time.

It processes **tokens**.

A token can be:

* a whole word
* part of a word
* punctuation
* whitespace-related text
* symbols

For example, something like:

```text
Spring AI is powerful
```

might conceptually become:

```text
["Spring", " AI", " is", " powerful"]
```

But don't memorize exact tokenization. Different models/tokenizers can tokenize the same text differently.

A longer word may be split:

```text
unbelievable
```

into pieces resembling:

```text
un + believable
```

Again, the exact split depends on the tokenizer.

---

# 4. Why do developers care about tokens?

Because **tokens are the fundamental unit of LLM context, generation limits, and often pricing**.

Suppose you have:

```text
System message
+
User message
+
Conversation history
+
Retrieved documents
+
Tool results
```

All of this consumes context.

Conceptually:

```text
┌─────────────────────────────┐
│ System message              │
│ User message                │
│ Conversation history        │
│ RAG context                 │
│ Tool results                │
│                             │
│          TOKENS             │
└─────────────────────────────┘
              ↓
             LLM
```

This becomes critical when you build RAG systems.

Suppose you retrieve 20 huge document chunks:

```text
Question
+
20 large chunks
+
conversation history
```

You might exceed the model's context capacity or unnecessarily increase latency/cost.

That's why later you'll study:

```text
Chunking
Context management
Memory
Summarization
Token limits
```

---

# 5. Context Window

This is another **extremely important concept**.

The **context window** is the amount of input/output context a model can handle for a request, measured in tokens.

Think of it as the model's working context.

For example:

```text
┌───────────────────────────────────────┐
│          Context Window               │
│                                       │
│ System instructions                   │
│ User message                          │
│ Conversation history                  │
│ Retrieved RAG documents               │
│ Tool results                          │
│ + generated output                    │
│                                       │
└───────────────────────────────────────┘
```

The exact limits depend on the model.

Don't think of context window as:

> "How much the model permanently remembers."

That's incorrect.

Instead:

> **Context window = how much information can participate in the current model interaction.**

---

# 6. Context window vs memory

This distinction becomes very important later.

Imagine you have a conversation:

```text
User:
My name is John.

Assistant:
Nice to meet you, John.

User:
What is my name?
```

How can the model answer?

Because the previous conversation may be included in the current request:

```text
System:
You are a helpful assistant.

User:
My name is John.

Assistant:
Nice to meet you, John.

User:
What is my name?
```

The model sees the conversation as context.

But if your application starts a completely new request:

```text
User:
What is my name?
```

the model doesn't automatically have to know the previous conversation.

That's why **application-level memory** becomes important.

Later in Spring AI you'll encounter things like:

```text
Chat Memory
Conversation ID
Message history
Memory Advisors
```

---

# 7. Prompts

A **prompt** is the input you provide to the model.

The simplest example:

```text
Explain dependency injection.
```

But a prompt can be much more complex:

```text
You are a senior Java developer.

Explain dependency injection in Spring Boot.

Requirements:
- Use simple language.
- Include a Java example.
- Explain constructor injection.
- Keep the answer under 500 words.
```

This contains instructions and context.

In Spring AI, you'll frequently construct prompts through abstractions such as:

```java
chatClient
    .prompt()
    .user("Explain dependency injection")
    .call();
```

The important thing at this stage is understanding that:

```text
Prompt
   ↓
Model
   ↓
Generated response
```

is the basic interaction.

---

# 8. System, User, and Assistant messages

This is particularly important for Spring AI.

Modern chat APIs generally represent conversations using different message roles.

Conceptually:

```text
SYSTEM
"You are a senior Java developer."

USER
"Explain Spring dependency injection."

ASSISTANT
"Dependency injection is..."
```

Let's understand each.

---

## System message

The system message provides high-level instructions/context for the model.

Example:

```text
You are a senior Java developer.

Always explain concepts using Java examples.
```

Then the user asks:

```text
What is dependency injection?
```

The model should use the system instructions while answering.

Think:

```text
System
  ↓
Defines behavior/instructions
```

In Spring AI you'll commonly see:

```java
.system("You are a senior Java developer")
```

---

# 9. User message

The user message represents the user's request.

Example:

```text
Explain dependency injection.
```

In Spring AI:

```java
.user("Explain dependency injection")
```

So conceptually:

```text
System:
"You are a senior Java developer."

User:
"Explain dependency injection."
```

---

# 10. Assistant message

The assistant message represents previous/generated model responses.

For example:

```text
User:
What is Spring?

Assistant:
Spring is a Java framework...

User:
What is dependency injection?
```

The previous assistant response can become part of the conversation context.

So a conversation can look like:

```text
SYSTEM
↓
USER
↓
ASSISTANT
↓
USER
↓
ASSISTANT
```

This is important when you later implement **chat history and memory**.

---

# 11. Chat Completion

You will hear the term **chat completion** frequently.

Conceptually:

```text
Messages
   ↓
Chat model
   ↓
Generated message
```

Example:

```text
System:
You are a Java expert.

User:
Explain interfaces in Java.
```

The model generates:

```text
Assistant:
An interface in Java defines...
```

That's a chat completion.

In a Spring AI application, conceptually:

```java
ChatResponse response =
    chatModel.call(prompt);
```

or through the higher-level:

```java
chatClient
    .prompt()
    .user("Explain Java interfaces")
    .call()
    .content();
```

You'll learn the actual Spring AI APIs in Phase 2.

For now, understand the underlying model interaction.

---

# 12. Temperature

Now we get into **model generation parameters**.

Temperature controls the randomness of token selection.

Imagine the model has:

```text
Java is a

language     0.60
platform     0.20
technology   0.10
framework    0.05
...
```

With a low temperature, the model tends to favor higher-probability choices.

With a higher temperature, it can explore lower-probability choices more often.

Conceptually:

```text
Low temperature
      ↓
More predictable
More consistent
Less variation
```

versus:

```text
High temperature
      ↓
More variation
More randomness
More creative possibilities
```

For example, for a factual extraction task:

```text
Temperature → low
```

might be preferable.

For creative writing:

```text
Temperature → higher
```

may be useful.

### Important misconception

Temperature does **not** mean:

> "Make the model smarter."

It primarily changes the randomness of generation.

---

# 13. Top-p

Top-p is another sampling parameter.

It is often called **nucleus sampling**.

Instead of considering every possible token, the model considers a subset whose combined probability reaches a certain threshold.

Imagine:

```text
Token probabilities:

A → 0.50
B → 0.25
C → 0.15
D → 0.05
E → 0.05
```

If:

```text
top-p = 0.90
```

the model might consider:

```text
A + B + C = 0.90
```

and sample from that group.

The exact behavior is model-dependent, but the conceptual idea is:

> **Top-p controls how large a probability mass of candidate tokens is considered during sampling.**

---

# 14. Temperature vs Top-p

For your roadmap, don't obsess over the mathematical details initially.

Remember:

```text
Temperature
    ↓
How random/diverse is sampling?

Top-p
    ↓
How much probability mass is considered?
```

In production, you generally shouldn't randomly tune both without understanding the model/provider's recommendations.

---

# 15. Model Parameters

When calling an LLM, you can provide parameters controlling generation.

Conceptually:

```json
{
  "model": "some-model",
  "temperature": 0.2,
  "max_tokens": 500,
  "top_p": 0.9
}
```

Think of the request as:

```text
              ┌────────────────────┐
Prompt ──────→│                    │
              │       LLM          │
Parameters ─→ │                    │
              └─────────┬──────────┘
                        ↓
                    Response
```

Common parameters include:

* model
* temperature
* top-p
* maximum output tokens
* stop sequences

The exact parameter names and supported features vary by provider/model.

---

# 16. Input tokens vs Output tokens

This is very important for backend development.

Suppose:

```text
Input:
"Explain Spring AI in detail..."
```

requires:

```text
1,000 input tokens
```

and the model generates:

```text
500 output tokens
```

Then conceptually:

```text
Input tokens  = 1,000
Output tokens =   500
Total         = 1,500
```

This matters for:

### Cost

Many model APIs price input and output separately.

### Context

Your input consumes context.

### Latency

Generating more output generally takes more time.

### Application design

If you're building RAG:

```text
Question
+
Retrieved chunks
+
History
+
System instructions
```

can become a very large input.

---

# 17. Maximum output tokens

You can generally constrain how much the model generates.

Conceptually:

```text
max output tokens = 300
```

means:

```text
Don't generate an unlimited response.
```

This is useful for APIs.

Imagine you're building:

```text
Resume parser
```

You don't want the model to produce a 10,000-token essay.

You want something like:

```json
{
  "name": "John",
  "skills": [
    "Java",
    "Spring Boot"
  ]
}
```

So output limits and structured generation become useful.

---

# 18. Streaming

Normally, you might receive the response after the model has generated it.

Without streaming:

```text
User
 ↓
Request
 ↓
LLM generates entire response
 ↓
Response
 ↓
UI
```

The user waits.

With streaming:

```text
User
 ↓
Request
 ↓
LLM
 ↓
"Spring"
 ↓
" AI"
 ↓
" is"
 ↓
" a"
 ↓
" framework"
 ↓
...
```

The application receives chunks as they become available.

So the UI can display:

```text
Spring
Spring AI
Spring AI is
Spring AI is a
Spring AI is a framework...
```

This is why streaming becomes its own phase in your roadmap.

Later you'll connect:

```text
Spring AI
   ↓
Flux<String>
   ↓
SSE
   ↓
React
```

---

# 19. Streaming does NOT mean the model thinks faster

This is an important distinction.

Streaming doesn't necessarily reduce the total generation time.

It reduces **time to first visible output**.

Without streaming:

```text
0s -------------------- 5s
                        ↓
                 Entire response
```

With streaming:

```text
0s
 ↓
First tokens

 ↓
More tokens

 ↓
More tokens

 ---------------------- 5s
 ↓
Complete response
```

The user gets feedback much earlier.

---

# 20. Embeddings

Now we reach the concept that connects LLM fundamentals to **RAG**.

An embedding model converts text into a numerical vector.

For example:

```text
"Spring Boot is a Java framework"
                 ↓
          Embedding Model
                 ↓
[0.12, -0.43, 0.81, 0.22, ...]
```

That vector represents semantic information about the text.

It isn't just a random list of numbers.

Texts with similar meanings tend to have embeddings that are relatively close in vector space.

For example:

```text
A:
"Spring Boot is used to build Java applications."

B:
"Spring Boot helps developers create Java applications."
```

Their embeddings may be relatively close.

Whereas:

```text
C:
"How to repair a bicycle"
```

would likely be much farther away.

---

# 21. Embeddings are NOT the same thing as chat generation

This distinction is critical.

You have two different model types:

```text
Chat Model
    ↓
Text → Generated text
```

and:

```text
Embedding Model
    ↓
Text → Vector
```

For example:

```text
Chat Model:

"Explain Spring Boot"
        ↓
"Spring Boot is..."
```

Embedding model:

```text
"Explain Spring Boot"
        ↓
[0.21, -0.44, 0.82, ...]
```

This distinction becomes fundamental when you start learning vector databases and RAG.

---

# 22. Why do embeddings exist?

Because computers can compare numerical vectors efficiently.

Imagine you have:

```text
Document 1 → vector A
Document 2 → vector B
Document 3 → vector C
Document 4 → vector D
```

User asks:

```text
"How do I configure Spring Security?"
```

You generate an embedding:

```text
Question
   ↓
Embedding model
   ↓
Question vector
```

Then compare it against document vectors:

```text
Question vector
       ↓
 ┌─────┼─────┬─────┐
 ↓     ↓     ↓     ↓
Doc A Doc B Doc C Doc D
```

The closest vectors represent potentially relevant documents.

That's the foundation of:

 of AI models"**, where we'll distinguish **ChatModel vs EmbeddingModel vs ImageModel vs AudioModel vs MultimodalModel vs RerankingModel**, and—most importantly—I'll show you **where each one fits in a real Spring Boot architecture**.

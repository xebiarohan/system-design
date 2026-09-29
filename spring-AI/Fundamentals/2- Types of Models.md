
# Types of AI Models

## What you should know by the end

You should be able to look at an AI architecture and say:

> "This part requires a Chat Model, this part requires an Embedding Model, this part uses a Reranker, and this part is Multimodal."

The six categories are:

```text
                    AI Models
                       │
       ┌───────────────┼────────────────┐
       │               │                │
       ▼               ▼                ▼
     Chat          Embedding          Image
       │               │                │
       └───────────────┼────────────────┘
                       │
              ┌────────┴────────┐
              ▼                 ▼
            Audio          Multimodal
                               
                       +
                       
                    Reranking
```

But there's an important nuance:

**These categories aren't always mutually exclusive.**

For example, a multimodal model can also behave as a chat model.

---

# Part 1 — Chat Models

### Time: ~20 minutes

Let's start with the model you'll interact with most often.

## What is a Chat Model?

A chat model is a model designed to take **messages/conversation context** as input and generate a response.

Conceptually:

```text
Messages
   │
   ▼
┌──────────────┐
│ Chat Model   │
└──────┬───────┘
       │
       ▼
   Response
```

For example:

```text
System:
You are a Java expert.

User:
Explain virtual threads.

        ↓

Chat Model

        ↓

Assistant:
Virtual threads are lightweight threads...
```

---

## Why "Chat" instead of "Text Generation"?

This is important.

Modern LLM APIs generally don't think only in terms of:

```text
String → String
```

They work with **messages**.

For example:

```text
[
  {
    role: "system",
    content: "You are a Java expert"
  },
  {
    role: "user",
    content: "Explain virtual threads"
  }
]
```

The model processes these messages and generates the next response.

This maps directly to concepts you'll encounter later in Spring AI:

```text
Prompt
Message
ChatModel
ChatResponse
ChatClient
```

But **don't study those APIs yet**.

That's Phase 1/2.

---

## What can a Chat Model do?

A chat model can:

* Answer questions
* Generate text
* Summarize
* Translate
* Generate code
* Extract information
* Reason over provided context
* Generate structured output
* Decide whether to call tools

For example:

```text
User
 ↓
"Find my order #1234"
 ↓
Chat Model
 ↓
Tool call
 ↓
Java application
 ↓
Order information
 ↓
Chat Model
 ↓
"Your order is arriving tomorrow."
```

Notice something important:

**The Chat Model itself doesn't necessarily have access to your database.**

Your application provides that capability through mechanisms such as tool calling.

You'll learn this much later in Phase 12.

---

# Part 2 — Embedding Models

### Time: ~25 minutes

This is arguably the **second most important model type for your roadmap**, because it becomes the foundation of:

```text
Embeddings
   ↓
Vector DB
   ↓
RAG
```

---

# What problem does an embedding model solve?

Suppose we have:

```text
A = "I love programming in Java."

B = "Java is my favorite programming language."

C = "The restaurant serves excellent food."
```

A human immediately knows:

```text
A ≈ B

A ≠ C
```

But computers need a numerical representation to perform mathematical comparisons.

That's what embeddings provide.

---

## Embedding

An embedding model converts content into a vector:

```text
Text
 ↓
Embedding Model
 ↓
Vector
```

For example:

```text
"I love programming in Java."

        ↓

[0.12, -0.45, 0.78, 0.21, ...]
```

The actual vector might have hundreds or thousands of dimensions.

Don't memorize the numbers.

The key idea is:

> **Embedding = numerical representation of semantic meaning.**

---

# Semantic similarity

Suppose:

```text
A = "How do I reset my password?"

B = "What is the procedure for changing my password?"

C = "How do I configure Kafka?"
```

After embedding:

```text
A → [vector A]
B → [vector B]
C → [vector C]
```

We expect:

```text
distance(A, B) → small
distance(A, C) → large
```

Therefore:

```text
A ≈ B
A ≠ C
```

This gives us **semantic search**.

---

# Why is this so important for your roadmap?

Look at your future phases:

```text
Phase 7
Embedding

      ↓

Phase 8
Vector Databases

      ↓

Phase 9
RAG
```

These three phases are basically one continuous story.

You'll eventually build:

```text
Document
   ↓
Chunk
   ↓
Embedding Model
   ↓
Vector
   ↓
Vector Database
```

Then:

```text
User Question
      ↓
Embedding Model
      ↓
Query Vector
      ↓
Vector Database
      ↓
Relevant Documents
      ↓
Chat Model
      ↓
Answer
```

So when you reach Phase 7, you'll already have the mental model.

---

# Chat Model vs Embedding Model

This distinction should become automatic for you.

|                   | Chat                   | Embedding              |
| ----------------- | ---------------------- | ---------------------- |
| Purpose           | Generate response      | Represent meaning      |
| Input             | Messages               | Text/data              |
| Output            | Text/structured data   | Vector                 |
| Generates answer? | Yes                    | No                     |
| Semantic search?  | Not its primary job    | Yes                    |
| RAG               | Generates final answer | Helps retrieve context |

Think:

```text
Chat Model:

Question
   ↓
Answer
```

versus:

```text
Embedding Model:

Text
   ↓
Vector
```

---

# Part 3 — Image Models

### Time: ~15 minutes

Image models operate on images.

There are several different capabilities.

---

## Text → Image

```text
"Generate a futuristic city"

          ↓

     Image Model

          ↓

       🖼 Image
```

This is image generation.

---

## Image → Image

For example:

```text
Existing image
      +
"Remove the background"
      ↓
Image Model
      ↓
Modified image
```

---

## Image → Text

A model can analyze an image and produce text:

```text
Image
 ↓
Model
 ↓
"There's a laptop on a desk."
```

But here's where terminology becomes important.

A model that can accept both **images and text** is generally considered **multimodal**.

So don't rigidly think:

```text
Image Model ≠ Multimodal Model
```

The capabilities can overlap.

---

# Part 4 — Audio Models

### Time: ~15 minutes

Audio models work with speech/audio.

The two most important transformations are:

```text
Speech → Text
```

and:

```text
Text → Speech
```

---

## Speech → Text

Also called **STT**.

```text
🎤 User speaks
      ↓
 Audio Model
      ↓
"Explain Kafka partitions"
```

Applications:

* Voice assistants
* Meeting transcription
* Call transcription
* Voice search

---

## Text → Speech

Also called **TTS**.

```text
"Your order has shipped."
          ↓
      Audio Model
          ↓
       🔊 Speech
```

---

# Part 5 — Multimodal Models

### Time: ~20 minutes

This is where modern AI gets particularly interesting.

A **multimodal model** can work with multiple types of information.

For example:

```text
Text
Image
Audio
Video
```

depending on the model.

---

## Example

Imagine you upload:

```text
invoice.jpg
```

and ask:

> "What is the total amount on this invoice?"

The model processes:

```text
Image + Text
       ↓
Multimodal Model
       ↓
"Total: ₹25,430"
```

---

## Another example

You give it a screenshot:

```text
Screenshot of Java exception
```

and ask:

> "What's wrong?"

The model can understand the visual content and respond.

---

# Multimodal vs Chat Model

This distinction is subtle and important.

Think about two different dimensions:

### Chat

Describes **how you interact with the model**.

```text
Conversation
   ↓
Chat model
```

### Multimodal

Describes **what types of information the model can process**.

```text
Text
Image
Audio
...
   ↓
Multimodal model
```

Therefore, a model can effectively be:

```text
Multimodal + Chat
```

For example:

```text
User:
[image]

"What is wrong with this code?"

        ↓

Multimodal Chat Model

        ↓

Answer
```

You'll see this become relevant in your **Phase 15 — Multimodal AI**.

---

# Part 6 — Reranking Models

### Time: ~20 minutes

This one is especially important because it connects directly to your **Phase 9 — RAG**.

Let's say the user asks:

> "How do I configure Kafka consumer retries?"

Your vector database performs similarity search and returns:

```text
Document A
Document B
Document C
Document D
...
Document T
```

These are candidates.

But perhaps:

```text
A → somewhat relevant
B → extremely relevant
C → somewhat relevant
D → irrelevant
...
```

We want to improve the ordering.

That's where a **reranking model** comes in.

---

# Retrieval vs Reranking

The architecture is:

```text
                    Query
                      │
                      ▼
              Embedding Model
                      │
                      ▼
               Vector Search
                      │
                      ▼
                Top 20 docs
                      │
                      ▼
              Reranking Model
                      │
                      ▼
                 Top 5 docs
                      │
                      ▼
                 Chat Model
                      │
                      ▼
                   Answer
```

This is a very common pattern in high-quality RAG systems.

---

# Why not just use the vector database?

Because the vector search and reranking stages have different jobs.

### Embedding + vector search

Optimized for:

> **Fast candidate retrieval**

### Reranker

Optimized for:

> **Better relevance ordering**

Think of it as:

```text
1,000,000 documents
        ↓
Fast search
        ↓
100 candidates
        ↓
Expensive but accurate reranker
        ↓
10 best documents
```

You wouldn't want to run the expensive reranker against one million documents.

---

# Putting All Six Together

Now let's create your mental map.

| Model              | Main responsibility            |
| ------------------ | ------------------------------ |
| 🧠 **Chat**        | Generate responses             |
| 📐 **Embedding**   | Convert meaning → vectors      |
| 🖼️ **Image**      | Generate/process images        |
| 🎙️ **Audio**      | Process/generate audio         |
| 👁️ **Multimodal** | Work with multiple modalities  |
| 🔎 **Reranking**   | Improve search relevance/order |

---

# The Most Important Architecture to Remember

This is the architecture you'll encounter repeatedly throughout your roadmap:

```text
                         User
                          │
                          ▼
                    User Question
                          │
                          ▼
                  ┌───────────────┐
                  │   Embedding   │
                  │     Model     │
                  └───────┬───────┘
                          │
                          ▼
                  Vector Database
                          │
                          ▼
                   Candidate Docs
                          │
                          ▼
                  ┌───────────────┐
                  │   Reranking   │
                  │     Model     │
                  └───────┬───────┘
                          │
                          ▼
                   Relevant Docs
                          │
                          ▼
                  ┌───────────────┐
                  │   Chat Model  │
                  └───────┬───────┘
                          │
                          ▼
                       Answer
```

And independently, you might have:

```text
Image + Text
     ↓
Multimodal Model
     ↓
Structured Answer
```

or:

```text
Speech
  ↓
Audio Model
  ↓
Text
  ↓
Chat Model
  ↓
Answer
```

---

# One More Important Concept: Models vs Providers

Since you're learning **Spring AI**, this distinction will save you a lot of confusion later.

Don't think:

```text
OpenAI = Chat Model
```

or:

```text
Anthropic = AI Model
```

Instead:

```text
                 AI Provider
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Chat       Embedding    Image
       Model        Model       Model
```

A provider can expose multiple model capabilities.

Spring AI's job is partly to give your application a consistent programming model across supported providers.

You'll study this properly in:

> **Phase 1 → Topic 3: What is Spring AI?**

---

# How This Maps to Your Roadmap

Your roadmap is actually structured quite nicely.

```text
Phase 0
   │
   ├── LLM fundamentals
   │
   └── Model types  ← YOU ARE HERE
            │
            ▼
Phase 1
   │
   ├── What is Spring AI?
   │
   └── Spring AI abstractions
            │
            ▼
Phase 2
   │
   └── Chat Models
            │
            ▼
Phase 7
   │
   └── Embeddings
            │
            ▼
Phase 8
   │
   └── Vector DB
            │
            ▼
Phase 9
   │
   └── RAG
            │
            ├── Retrieval
            ├── Reranking
            │
            ▼
Phase 15
   │
   └── Multimodal AI
```

So **don't worry that we haven't gone deep into reranking, multimodal, etc. yet**.

Your roadmap intentionally revisits these concepts later at the appropriate depth.

---

# What I Want You to Remember From Topic 2

If I asked you in an interview:

### "What's an embedding model?"

You should be able to say:

> An embedding model converts data into numerical vectors that represent semantic meaning. These vectors can be compared to perform semantic similarity search, which is commonly used with vector databases and RAG.

### "What's a reranking model?"

> A reranking model takes a query and a set of retrieved candidate documents and reorders them according to their relevance. It's often used after vector search to improve retrieval quality.

### "What's a multimodal model?"

> A multimodal model can process multiple types of data, such as text, images, audio, or video, depending on its capabilities.

### "What's the difference between Chat and Embedding models?"

> A chat model generates responses, while an embedding model converts information into vectors representing semantic meaning.

Those four answers are the core of this topic.

---

## 🧪 Quick Test

Before moving to **Topic 3 — What is Spring AI?**, try answering these without looking back:

1. Why can't we simply use a Chat Model for semantic search?
2. What does an embedding vector represent?
3. Why do we need a vector database after generating embeddings?
4. Why might a RAG system use both an embedding model **and** a reranking model?
5. What's the difference between a Chat Model and a Multimodal Model?
6. If a user speaks into your application and you want the LLM to answer, what model types could be involved?
7. In this pipeline, identify every model:

```text
User speaks
   ↓
???
   ↓
Text
   ↓
???
   ↓
Relevant documents
   ↓
???
   ↓
Answer
```

If you can answer those confidently, **Topic 2 is complete**, and we can move to **Phase 1 → Topic 3: What is Spring AI?**

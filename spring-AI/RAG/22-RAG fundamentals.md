# 27. RAG Fundamentals

## 1. What problem does RAG solve?

An LLM has a few important limitations:

### Problem 1 — It doesn't know your private data

Suppose your company has:

```text
HR_Policy.pdf
```

containing:

> Employees can work remotely up to 3 days per week.

Your LLM doesn't automatically know this document.

You could put the entire document into the prompt:

```text
User:
Can I work remotely 3 days a week?

System:
Here is the entire HR policy:
[huge document...]

Answer the question.
```

This has problems:

* Large prompt
* Expensive
* Slow
* Context-window limitations
* You can't realistically send thousands of documents every time

---

### Problem 2 — LLM knowledge can be outdated

Imagine your database contains:

```text
Product:
Spring AI 2.0
```

The model may not have reliable knowledge about your latest internal information.

---

### Problem 3 — Hallucination

If the LLM doesn't know the answer, it may generate something that **sounds correct but isn't actually supported by your data**.

RAG addresses these problems.

---

# 2. What is RAG?

**RAG = Retrieval-Augmented Generation**

Break the name apart:

### Retrieval

Find relevant information from your own knowledge source.

### Augmented

Add that information to the LLM's prompt.

### Generation

Let the LLM generate an answer using that retrieved information.

So the basic idea is:

```text
User Question
      ↓
Retrieve relevant information
      ↓
Add information to prompt
      ↓
LLM
      ↓
Answer
```

---

# 3. Simple example

Suppose you have:

```text
HR Policy

Employees can work remotely up to 3 days per week.
Remote work must be approved by the manager.
Employees must be available during core working hours.
```

User asks:

```text
How many days can I work remotely?
```

RAG searches your knowledge base and retrieves:

```text
Employees can work remotely up to 3 days per week.
```

Then it effectively sends the LLM something like:

```text
System:
Answer the question using the following context.

Context:
Employees can work remotely up to 3 days per week.

Question:
How many days can I work remotely?
```

LLM:

```text
Employees can work remotely up to 3 days per week.
```

The important point is:

> **The LLM didn't need to know the HR policy beforehand.**

RAG supplied the relevant knowledge at runtime.

---

# 4. RAG architecture

A typical RAG system looks like this:

```text
                    ┌──────────────────┐
                    │   Documents      │
                    │ PDF / DOC / DB   │
                    └────────┬─────────┘
                             ↓
                       Text Extraction
                             ↓
                         Chunking
                             ↓
                        Embeddings
                             ↓
                    ┌──────────────────┐
                    │  Vector Store    │
                    │ pgvector / etc.  │
                    └────────┬─────────┘
                             │
                             │
                    ┌────────▼─────────┐
User Question ────►│     Retrieval     │
                    └────────┬─────────┘
                             ↓
                    Relevant Chunks
                             ↓
                    ┌──────────────────┐
                    │ Prompt + Context │
                    └────────┬─────────┘
                             ↓
                         LLM / Chat Model
                             ↓
                          Answer
```

There are really **two major phases**.

---

# 5. Phase 1 — Indexing / Ingestion

Before users ask questions, you need to prepare your documents.

For example:

```text
HR Policy.pdf
```

goes through:

```text
PDF
 ↓
Extract text
 ↓
Split into chunks
 ↓
Generate embeddings
 ↓
Store vectors
```

You've already studied most of these pieces.

---

## Step 1 — Load the document

Spring AI can read documents using document readers.

You've already used:

```java
TikaDocumentReader
```

For example:

```text
HR_Policy.pdf
        ↓
Document
```

The document contains textual content plus metadata.

---

# 6. Step 2 — Chunking

You normally don't embed an entire 50-page PDF as one vector.

Instead:

```text
HR Policy
    ↓
Chunk 1
Chunk 2
Chunk 3
Chunk 4
...
```

For example:

```text
Chunk 1:
Employees are entitled to...

Chunk 2:
Remote work is permitted...

Chunk 3:
Employees must submit...

Chunk 4:
Leave policy requires...
```

Why?

Because when the user asks:

```text
How many days can I work remotely?
```

you don't want to retrieve the entire 50-page document.

You want something like:

```text
Chunk 2
```

---

# 7. Step 3 — Generate embeddings

Each chunk is converted into an embedding.

For example:

```text
"Employees can work remotely up to 3 days per week."
```

becomes something conceptually like:

```text
[0.12, -0.43, 0.78, 0.21, ...]
```

That vector represents the **semantic meaning** of the text.

You learned this in your previous topic.

---

# 8. Step 4 — Store the embeddings

The chunks and embeddings are stored in a vector database.

For example:

```text
PostgreSQL + pgvector
```

Conceptually:

| ID | Content                   | Embedding    |
| -- | ------------------------- | ------------ |
| 1  | Remote work is allowed... | `[0.12,...]` |
| 2  | Employees must submit...  | `[0.44,...]` |
| 3  | Annual leave...           | `[0.81,...]` |

Now your knowledge base is ready.

---

# 9. Phase 2 — Retrieval + Generation

This happens when the user actually asks a question.

Suppose:

```text
How many days can I work remotely?
```

---

## Step 1 — Embed the question

The question is converted into an embedding:

```text
"How many days can I work remotely?"
                    ↓
             Query embedding
```

---

## Step 2 — Similarity search

The query embedding is compared with vectors stored in your vector database.

Conceptually:

```text
Query
 ↓
Embedding
 ↓
Vector similarity search
 ↓
Top matching chunks
```

Maybe the database returns:

```text
Chunk 27:
Employees can work remotely up to 3 days per week.

Chunk 31:
Remote work requires manager approval.

Chunk 35:
Employees must remain available during core hours.
```

These are the **retrieved documents/chunks**.

---

# 10. Step 3 — Augment the prompt

Now Spring AI builds a prompt containing the retrieved context.

Conceptually:

```text
System:
You are an HR assistant.

Use the following context to answer the question.

<context>
Employees can work remotely up to 3 days per week.

Remote work requires manager approval.

Employees must remain available during core hours.
</context>

Question:
How many days can I work remotely?
```

The important part is:

```text
Question
+
Retrieved Context
```

---

# 11. Step 4 — LLM generates the answer

The LLM receives:

```text
Question
+
Relevant company information
```

and produces:

```text
You can work remotely up to 3 days per week,
subject to manager approval.
```

That's the **Generation** part of RAG.

---

# 12. RAG vs normal LLM

This distinction is important.

### Normal LLM

```text
User
 ↓
LLM
 ↓
Answer
```

The LLM primarily relies on what it learned during training plus the current conversation/prompt.

### RAG

```text
User
 ↓
Retriever
 ↓
Relevant knowledge
 ↓
LLM
 ↓
Answer
```

The LLM gets **external knowledge at query time**.

---

# 13. RAG vs fine-tuning

This is another very important distinction.

People often think:

> "I have company documents, so I should train/fine-tune the model."

Usually, that's **not the first solution**.

### RAG

You keep the knowledge outside the model:

```text
Documents
   ↓
Vector DB
   ↓
Retrieve
   ↓
LLM
```

Good for:

* Company documentation
* HR policies
* Product documentation
* Frequently changing information
* FAQs
* Technical documentation

---

### Fine-tuning

You modify the model's behavior/weights using training data.

Conceptually:

```text
Training data
     ↓
Fine-tuning
     ↓
Model
```

Fine-tuning is more appropriate when you want to change **how the model behaves**, rather than simply give it a large collection of changing facts.

A useful mental model:

> **RAG gives the model knowledge. Fine-tuning changes the model's behavior.**

---

# 14. RAG doesn't mean "just vector search"

This is an important point for your roadmap.

A basic RAG pipeline is:

```text
Documents
 ↓
Chunk
 ↓
Embedding
 ↓
Vector Store
 ↓
Retrieve
 ↓
Prompt
 ↓
LLM
```

But real-world RAG can become considerably more sophisticated.

For example:

```text
User question
      ↓
Query transformation
      ↓
Hybrid retrieval
      ↓
Metadata filtering
      ↓
Reranking
      ↓
Context selection
      ↓
Prompt
      ↓
LLM
```

You'll encounter these concepts later.

---

# 15. What exactly is the "Retriever"?

The **retriever** is the component responsible for finding relevant information.

Conceptually:

```java
List<Document> documents =
        retriever.retrieve(query);
```

Input:

```text
"How many days can I work remotely?"
```

Output:

```text
[
    Document("Employees can work remotely up to 3 days..."),
    Document("Remote work requires manager approval...")
]
```

The retriever doesn't generate the answer.

It only answers:

> **"Which pieces of knowledge are relevant to this question?"**

The LLM then answers:

> **"Given those pieces of knowledge, what should I say?"**

---

# 16. Retriever vs Vector Store

These are related but different.

### Vector Store

Stores and searches vectors.

```text
Vector Store
    ↓
similaritySearch()
```

### Retriever

Provides a higher-level retrieval abstraction.

```text
Retriever
    ↓
search knowledge
    ↓
return relevant Documents
```

So:

```text
Retriever
    ↓
Vector Store
    ↓
Similarity Search
```

is a common architecture.

---

# 17. What does Spring AI add?

Spring AI provides abstractions that make implementing RAG easier.

You've already learned:

```text
EmbeddingModel
VectorStore
Document
```

Spring AI can connect these pieces.

Conceptually:

```text
Document
   ↓
EmbeddingModel
   ↓
VectorStore
```

and during a query:

```text
User Question
      ↓
VectorStore similarity search
      ↓
Documents
      ↓
Prompt
      ↓
ChatModel
```

---

# 18. A simple Spring AI RAG example

A conceptual implementation might look like:

```java
List<Document> documents =
        vectorStore.similaritySearch(
                SearchRequest.builder()
                        .query(question)
                        .topK(5)
                        .build()
        );
```

Suppose the result is:

```text
Document 1
Document 2
Document 3
```

You can then construct a prompt:

```java
String context = documents.stream()
        .map(Document::getText)
        .collect(Collectors.joining("\n"));
```

Then:

```java
String prompt = """
        Answer the question using the following context.

        Context:
        %s

        Question:
        %s
        """.formatted(context, question);
```

And call the model:

```java
String answer = chatClient
        .prompt()
        .user(prompt)
        .call()
        .content();
```

That's essentially a **basic manual RAG implementation**.

---

# 19. Where does your pgvector knowledge fit?

Your previous topic:

```text
PostgreSQL + pgvector
```

fits directly into RAG.

For example:

```text
                    RAG
                     │
        ┌────────────┴────────────┐
        │                         │
    Retrieval                  Generation
        │                         │
   Vector Store                 LLM
        │
 PostgreSQL
    + pgvector
```

So your learning progression is actually very logical:

```text
Embeddings
     ↓
Vector Databases
     ↓
Spring AI VectorStore
     ↓
PostgreSQL + pgvector
     ↓
RAG Fundamentals
```

Now you're combining everything you've learned.

---

# 20. Important RAG terminology

You should be comfortable with these terms.

### Corpus

The entire collection of knowledge.

```text
HR policies
+
Technical documentation
+
Product manuals
+
FAQs
```

---

### Document

An individual knowledge item.

```text
HR_Policy.pdf
```

---

### Chunk

A smaller section of a document.

```text
Page 12 → Chunk 37
```

---

### Embedding

Numerical representation of semantic meaning.

```text
Text
 ↓
Embedding
 ↓
Vector
```

---

### Vector Store

Database/storage system for embeddings and associated documents.

```text
pgvector
Pinecone
Weaviate
Milvus
etc.
```

---

### Retriever

Finds relevant documents/chunks.

---

### Context

The retrieved information given to the LLM.

---

### Generation

The LLM's final answer based on the question + retrieved context.

---

# 21. The most important mental model

Remember this:

```text
                 KNOWLEDGE BASE
                       │
                ┌──────▼──────┐
                │  Documents  │
                └──────┬──────┘
                       ↓
                    Chunking
                       ↓
                   Embeddings
                       ↓
                 ┌─────────────┐
                 │ Vector Store│
                 └──────┬──────┘
                        │
                        │
User Question ──────────┤
                        ↓
                   Retrieval
                        ↓
                Relevant Chunks
                        ↓
                ┌───────────────┐
                │ Prompt +      │
                │ Retrieved     │
                │ Context       │
                └───────┬───────┘
                        ↓
                       LLM
                        ↓
                     Answer
```

Or, in one sentence:

> **RAG retrieves relevant external knowledge at runtime and gives it to the LLM as context so the LLM can generate a grounded answer.**

---

# 22. One subtle but very important point

RAG **doesn't guarantee correctness**.

Imagine your vector search retrieves the wrong chunks:

```text
Question:
How many days can I work remotely?

Retrieved:
Annual leave policy
Sick leave policy
Travel policy
```

The LLM still has to answer.

It may:

```text
"I don't have enough information..."
```

or potentially hallucinate.

Therefore, RAG quality depends heavily on:

```text
Document quality
      +
Chunking quality
      +
Embedding quality
      +
Retrieval quality
      +
Prompt quality
      +
LLM quality
```

This is why later RAG topics become important:

```text
Basic RAG
   ↓
Better retrieval
   ↓
Metadata filtering
   ↓
Hybrid search
   ↓
Reranking
   ↓
Query transformation
   ↓
Evaluation
```

---

## 23. Your HR PDF example

The example you've already worked with is basically a RAG ingestion pipeline:

```text
HR_Policy.pdf
      ↓
TikaDocumentReader
      ↓
Document
      ↓
TokenTextSplitter
      ↓
Chunks
      ↓
EmbeddingModel
      ↓
PostgreSQL + pgvector
```

Then at question time:

```text
"What is the company's remote work policy?"
                 ↓
             Embedding
                 ↓
          pgvector search
                 ↓
        Relevant HR chunks
                 ↓
             ChatClient
                 ↓
              Answer
```

So **RAG Fundamentals is the topic where all your previous vector/embedding knowledge finally comes together into an actual AI application.**

### The key flow to memorize

```text
INGESTION

Documents
 → Load
 → Chunk
 → Embed
 → Store


QUERY

Question
 → Embed
 → Retrieve
 → Add context to prompt
 → LLM
 → Answer
```

That is the foundation. Everything you learn after Topic 27 will mostly be about **making this pipeline more accurate, efficient, scalable, and reliable**.

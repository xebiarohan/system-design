# 24. Spring AI `VectorStore`

## 1. The problem Spring AI solves

From the previous topic, you know that a vector database needs to do things like:

```text
Document
   ↓
Embedding
   ↓
Store vector
   ↓
Similarity search
   ↓
Return relevant documents
```

Without Spring AI, your application would need to know the APIs of whichever vector database you're using.

For example:

```text
Spring Boot
    ↓
PostgreSQL + pgvector API
```

If you later switch to another vector database:

```text
Spring Boot
    ↓
Pinecone API
```

your application code could require significant changes.

Spring AI provides an abstraction:

```text
Spring Boot
    ↓
Spring AI VectorStore
    ↓
Vector Database
```

So your Java application primarily works with **`VectorStore`**, rather than directly depending on a particular vector database implementation.

---

# 2. The `VectorStore` abstraction

The central interface is:

```java
VectorStore
```

Conceptually, it provides operations such as:

```text
add documents
delete documents
search documents
```

The important mental model is:

```text
                    Spring AI
                       │
                       ▼
                  VectorStore
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       PgVector       Redis      Pinecone
```

Your application talks to:

```java
VectorStore
```

The underlying implementation handles communication with the actual vector database.

---

# 3. The `Document` object

Before using a `VectorStore`, you need to understand the object that you're storing.

Spring AI represents a piece of information using a `Document`.

Conceptually:

```text
Document
├── id
├── text/content
└── metadata
```

For example:

```java
Document document = new Document(
    "Spring Boot makes it easy to build Java applications.",
    Map.of(
        "category", "spring",
        "source", "spring-guide"
    )
);
```

Think of this as:

```text
┌─────────────────────────────────────────┐
│ Document                                │
├─────────────────────────────────────────┤
│ Content:                                │
│ "Spring Boot makes it easy..."          │
│                                         │
│ Metadata:                               │
│ category = spring                       │
│ source   = spring-guide                 │
└─────────────────────────────────────────┘
```

The embedding isn't something you normally have to manually calculate before calling `VectorStore`.

That's an important convenience.

---

# 4. Adding documents

Suppose you have:

```java
Document document =
        new Document("Spring Boot is a Java framework.");
```

You can add it to the vector store:

```java
vectorStore.add(List.of(document));
```

Conceptually, Spring AI does something like:

```text
Document
   │
   ▼
EmbeddingModel
   │
   ▼
Vector
   │
   ▼
VectorStore
   │
   ▼
Vector Database
```

So you don't necessarily need to write:

```java
float[] vector = embeddingModel.embed(...);
```

and then manually insert that vector into the database.

The `VectorStore` integrates the embedding step with storage.

---

# 5. Very important: `VectorStore` + `EmbeddingModel`

This is one of the most important relationships to understand.

You learned earlier:

```text
EmbeddingModel
```

converts text into vectors.

Now:

```text
VectorStore
```

stores and searches those vectors.

So:

```text
EmbeddingModel
     │
     │ converts
     ▼
Text → Vector

VectorStore
     │
     │ stores/searches
     ▼
Vector Database
```

Together:

```text
             Spring AI
                 │
       ┌─────────┴─────────┐
       ▼                   ▼
EmbeddingModel          VectorStore
       │                   │
       ▼                   ▼
    vectors          store/search
```

---

# 6. Searching the VectorStore

This is where things become interesting.

Suppose you have stored:

```text
Document 1:
"Spring Boot is a Java framework."

Document 2:
"React is a JavaScript library."

Document 3:
"Spring AI provides abstractions for AI applications."
```

Now the user asks:

> "How do I build AI applications using Java?"

You perform a similarity search.

Conceptually:

```java
List<Document> results =
        vectorStore.similaritySearch(
            "How do I build AI applications using Java?"
        );
```

Spring AI will essentially perform:

```text
User Query
    ↓
EmbeddingModel
    ↓
Query Vector
    ↓
VectorStore
    ↓
Similarity Search
    ↓
Relevant Documents
```

For example:

```text
Document 3 → highly relevant
Document 1 → relevant
Document 2 → not very relevant
```

---

# 7. Top-K search

Usually you don't want every matching document.

You want:

> "Give me the top 5 most relevant documents."

This is **Top-K**.

For example:

```java
SearchRequest request = SearchRequest
        .query("How do I build AI applications using Java?")
        .topK(5);
```

Then:

```java
List<Document> results =
        vectorStore.similaritySearch(request);
```

Conceptually:

```text
100,000 documents
       ↓
Vector similarity search
       ↓
Rank by similarity
       ↓
Top 5
       ↓
Return
```

This is the same **Top-K** concept you learned in Topic 23.

---

# 8. Similarity threshold

Top-K alone isn't always enough.

Imagine you ask:

> "How do I configure Kubernetes?"

But your database contains no Kubernetes documentation.

If you simply say:

```text
Top-K = 5
```

the vector database may still return five documents that are the *least bad matches*.

That's potentially dangerous for RAG.

Instead, you can use a similarity threshold.

Conceptually:

```text
Query
 ↓
Similarity Search
 ↓
Only results with similarity >= threshold
```

For example:

```text
threshold = 0.75
```

Results:

```text
Document A → 0.94 ✅
Document B → 0.88 ✅
Document C → 0.79 ✅
Document D → 0.52 ❌
Document E → 0.31 ❌
```

This helps prevent irrelevant context from being sent to the LLM.

---

# 9. `SearchRequest`

The search request is useful because you can express search configuration together.

Conceptually:

```java
SearchRequest request = SearchRequest
        .query("How do I reset my password?")
        .topK(5)
        .similarityThreshold(0.7);
```

Then:

```java
List<Document> documents =
        vectorStore.similaritySearch(request);
```

So instead of thinking:

```java
vectorStore.search(...)
```

as just a simple database query, think:

```text
SearchRequest
├── query
├── topK
├── similarity threshold
└── metadata filter
```

This becomes particularly important when you reach **metadata filtering**.

---

# 10. Metadata filtering

Suppose your vector store contains:

```text
Document A
metadata:
department = IT

Document B
metadata:
department = HR

Document C
metadata:
department = Finance
```

The user asks:

> "What is the password reset procedure?"

You may want:

```text
department == IT
```

in addition to semantic similarity.

Conceptually:

```java
SearchRequest request = SearchRequest
        .query("What is the password reset procedure?")
        .topK(5)
        .filterExpression("department == 'IT'");
```

The actual filter-expression syntax can depend on the Spring AI version/vector-store implementation, so when you implement this, use the version-specific Spring AI API you're working with.

The conceptual flow is the important part:

```text
                Query
                  │
                  ▼
          Semantic Search
                  │
                  +
          Metadata Filter
                  │
                  ▼
                Top-K
```

Your roadmap explicitly calls this combination out in Topic 26. 

---

# 11. Why `Document` is so important for RAG

Remember the RAG architecture from your roadmap:

```text
PDF
 ↓
Document Reader
 ↓
Chunking
 ↓
Embedding
 ↓
PostgreSQL + pgvector
```



The pieces you are learning now fit directly into that pipeline.

Suppose you have:

```text
company-policy.pdf
```

You read it and split it into chunks:

```text
Chunk 1
"Employees are entitled to..."

Chunk 2
"Employees must request leave..."

Chunk 3
"Managers must approve..."
```

Each chunk becomes a Spring AI `Document`.

```text
PDF
 ↓
Document
 ↓
Document
 ↓
Document
```

Then:

```java
vectorStore.add(documents);
```

The documents are embedded and stored.

Later:

```java
vectorStore.similaritySearch(...)
```

retrieves the relevant chunks.

---

# 12. Full example

Imagine you're building a company knowledge assistant.

### Step 1 — Create documents

```java
List<Document> documents = List.of(
    new Document(
        "Employees get 25 days of annual leave.",
        Map.of("department", "HR")
    ),

    new Document(
        "Employees can reset their password using the IT portal.",
        Map.of("department", "IT")
    ),

    new Document(
        "VPN access requires multi-factor authentication.",
        Map.of("department", "IT")
    )
);
```

### Step 2 — Store them

```java
vectorStore.add(documents);
```

Conceptually:

```text
Documents
    ↓
EmbeddingModel
    ↓
Vectors
    ↓
VectorStore
    ↓
Vector Database
```

### Step 3 — User asks a question

```text
"How can I reset my password?"
```

### Step 4 — Search

```java
SearchRequest request = SearchRequest
        .query("How can I reset my password?")
        .topK(3);

List<Document> results =
        vectorStore.similaritySearch(request);
```

### Step 5 — Results

You might get:

```text
1. "Employees can reset their password using the IT portal."
2. "VPN access requires multi-factor authentication."
3. ...
```

Those results can now become context for the LLM.

---

# 13. VectorStore does NOT generate the final answer

This distinction is really important.

`VectorStore` is **not an LLM**.

It doesn't answer:

> "How do I reset my password?"

Instead, it retrieves information:

```text
VectorStore
     ↓
Relevant documents
```

Then:

```text
Relevant documents
        +
User question
        ↓
       LLM
        ↓
     Answer
```

So:

```text
VectorStore → Retrieve
LLM         → Generate
```

That's the fundamental RAG architecture.

---

# 14. `VectorStore` vs `EmbeddingModel`

It's easy to mix these up.

### `EmbeddingModel`

Answers:

> "How do I convert this text into a vector?"

```text
Text
 ↓
EmbeddingModel
 ↓
Vector
```

### `VectorStore`

Answers:

> "Where do I store these vectors, and how do I find similar ones?"

```text
Vector
 ↓
VectorStore
 ↓
Store/Search
```

Together:

```text
             Text
              │
              ▼
       ┌──────────────┐
       │ EmbeddingModel│
       └──────────────┘
              │
              ▼
           Vector
              │
              ▼
       ┌──────────────┐
       │  VectorStore │
       └──────────────┘
              │
              ▼
       Vector Database
```

---

# 15. Where PostgreSQL + pgvector comes in

You might be wondering:

> "If Spring AI gives me `VectorStore`, why do I need pgvector?"

Because `VectorStore` is an **abstraction**, not the actual database.

You'll eventually have:

```text
Spring Boot
     ↓
Spring AI
     ↓
VectorStore
     ↓
PgVectorStore
     ↓
PostgreSQL
     +
pgvector
```

The next topic in your roadmap is specifically **PostgreSQL + pgvector**, and the roadmap recommends starting there. 

---

# 16. The architecture you should remember

At this stage, keep this diagram in your head:

```text
                 SPRING AI
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
   EmbeddingModel          VectorStore
          │                     │
          │                     │
          ▼                     ▼
      Embeddings        Store / Search
                                │
                                ▼
                        Vector Database
                                │
                                ▼
                     PostgreSQL + pgvector
```

And during retrieval:

```text
User Question
      │
      ▼
EmbeddingModel
      │
      ▼
Query Vector
      │
      ▼
VectorStore
      │
      ├── similarity search
      ├── top-K
      └── metadata filter
      │
      ▼
Relevant Documents
```

---

# 17. The key APIs to remember

Don't try to memorize every method yet. At this point, focus on these concepts:

```java
VectorStore
```

and:

```java
vectorStore.add(documents);
```

for storing documents.

And:

```java
vectorStore.similaritySearch(...)
```

for retrieving relevant documents.

And understand the search configuration:

```text
SearchRequest
 ├── query
 ├── topK
 ├── similarity threshold
 └── metadata filter
```

That's enough for Topic 24.

---

# 18. How Topic 23 → 24 → 25 connect

This sequence is particularly important:

```text
Topic 23
Vector Databases
      │
      │ understand the concept
      ▼
Topic 24
Spring AI VectorStore
      │
      │ understand the Java abstraction
      ▼
Topic 25
PostgreSQL + pgvector
      │
      │ understand the actual implementation
      ▼
Topic 26
Metadata Filtering
      │
      ▼
RAG
```

Your roadmap intentionally separates **the concept**, **the Spring abstraction**, and **the concrete database implementation**. 

### 🧠 One sentence to remember

> **`EmbeddingModel` turns text into vectors; `VectorStore` stores those vectors and retrieves semantically similar `Document`s; the LLM then uses those retrieved documents to generate the answer.**

And honestly, once you understand that sentence deeply, you're about **80% of the way to understanding the mechanics of basic RAG**.

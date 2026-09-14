# 30. RAG Retrieval

## 1. What is retrieval?

Suppose your vector database contains 10,000 chunks.

The user asks:

```text
How many days can I work remotely?
```

You obviously don't want to send all 10,000 chunks to the LLM.

Instead:

```text
10,000 chunks
      ↓
   Retrieval
      ↓
Top relevant chunks
      ↓
     LLM
```

So retrieval means:

> **Finding the most relevant pieces of information from your knowledge base for a user's question.**

---

# 2. Where retrieval sits in RAG

Your complete RAG flow now becomes:

```text
                INGESTION
                    ↓
Documents → Chunks → Embeddings
                    ↓
              Vector Store
                    │
                    │
                    ↓
User Question → Retrieval
                    ↓
             Relevant Chunks
                    ↓
             Prompt + Context
                    ↓
                   LLM
                    ↓
                 Answer
```

The retrieval stage is therefore the bridge between:

```text
Vector Database
        ↓
      LLM
```

---

# 3. Basic retrieval using similarity search

You already learned embeddings.

Suppose we have:

```text
Question:
"What is the remote work policy?"
```

First, the question is converted into an embedding:

```text
Question
   ↓
Embedding
   ↓
[0.12, -0.45, 0.78, ...]
```

Your vector database contains:

```text
Chunk A → [0.10, -0.43, 0.75, ...]
Chunk B → [0.80,  0.11, 0.22, ...]
Chunk C → [0.13, -0.40, 0.77, ...]
```

Similarity is calculated between the query vector and stored vectors.

Conceptually:

```text
Query
 │
 ├── Chunk A → 0.91 similarity
 ├── Chunk B → 0.31 similarity
 └── Chunk C → 0.88 similarity
```

So:

```text
Chunk A
Chunk C
```

are good candidates for retrieval.

---

# 4. What is `topK`?

This is one of the most important retrieval parameters.

**`topK` means:**

> How many of the highest-ranking matching chunks should be returned?

For example:

```text
topK = 3
```

means:

```text
Query
 ↓
Similarity Search
 ↓
Top 3 results
```

Suppose the database returns:

```text
1. Chunk A → 0.95
2. Chunk B → 0.91
3. Chunk C → 0.87
4. Chunk D → 0.65
5. Chunk E → 0.42
```

With:

```text
topK = 3
```

you get:

```text
Chunk A
Chunk B
Chunk C
```

---

# 5. Why not `topK = 1`?

You might think:

> "Just get the single most similar chunk."

Sometimes that works.

But consider:

```text
Chunk 1:
Employees can work remotely up to 3 days per week.

Chunk 2:
Remote work requires manager approval.

Chunk 3:
Employees must remain available during core working hours.
```

The user's question might be:

> What are the rules for working remotely?

You need **all three** pieces.

So:

```text
topK = 1
```

might retrieve only:

```text
Employees can work remotely up to 3 days per week.
```

The LLM misses the approval requirement.

With:

```text
topK = 3
```

you potentially get the complete context.

---

# 6. Why not use a huge `topK`?

Suppose:

```text
topK = 100
```

Now you send 100 chunks to the LLM.

Problems:

### More tokens

```text
100 chunks
   ↓
Large prompt
   ↓
Higher cost
```

### More irrelevant information

```text
Relevant chunks
+
Somewhat relevant chunks
+
Irrelevant chunks
```

The LLM now has to figure out what's important.

### Potentially worse answers

More context doesn't automatically mean better answers.

You can get **context noise**.

So the goal is:

> **Retrieve enough information to answer the question, but avoid unnecessary information.**

---

# 7. `topK` is not the same as "number of correct results"

This is subtle.

Suppose:

```text
topK = 5
```

The vector database will generally return **up to five highest-ranking candidates**, even if some aren't very relevant.

For example:

```text
Chunk A → 0.92
Chunk B → 0.88
Chunk C → 0.82
Chunk D → 0.31
Chunk E → 0.19
```

If you simply use `topK = 5`, you could receive D and E even though they aren't useful.

This is where the **similarity threshold** becomes important.

---

# 8. Similarity threshold

A similarity threshold says:

> **Only return results whose similarity is good enough.**

For example:

```text
threshold = 0.75
```

Given:

```text
Chunk A → 0.92
Chunk B → 0.88
Chunk C → 0.82
Chunk D → 0.31
Chunk E → 0.19
```

you keep:

```text
Chunk A
Chunk B
Chunk C
```

and discard:

```text
Chunk D
Chunk E
```

So:

```text
topK
```

controls **how many results you're willing to retrieve**.

While:

```text
threshold
```

controls **how relevant a result must be to qualify**.

---

# 9. `topK` vs threshold

Think of it like this:

```text
                    Retrieval
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
           topK              threshold
             │                   │
     Maximum number       Minimum relevance
       of results            required
```

Example:

```text
topK = 5
threshold = 0.75
```

means:

> Return at most 5 results, but only if they have similarity ≥ 0.75.

If only three satisfy the threshold:

```text
Result 1 → 0.94
Result 2 → 0.89
Result 3 → 0.81
Result 4 → 0.52
Result 5 → 0.41
```

you get only:

```text
3 results
```

not five.

---

# 10. Important caveat: similarity scores aren't universally comparable

Don't assume:

```text
0.80 = always highly relevant
```

across every vector store, embedding model, or similarity metric.

Different systems can use:

* cosine similarity
* dot product
* Euclidean distance
* other metrics

And the score ranges/meaning can differ.

So a threshold such as:

```text
0.75
```

is **not a magic universal value**.

You should tune it using your own data.

---

# 11. Metadata filtering

Now we get to a very powerful retrieval technique.

Suppose your vector store contains:

```text
HR documents
Engineering documents
Finance documents
Security documents
```

User asks:

> What is the employee remote work policy?

You could first filter:

```text
department = HR
```

Then perform similarity search.

Conceptually:

```text
All 10,000 chunks
        ↓
Metadata filter
department = HR
        ↓
2,000 chunks
        ↓
Similarity search
        ↓
Top relevant chunks
```

This is called **metadata filtering**.

---

# 12. Why filtering is useful

Imagine two documents contain the phrase:

```text
"remote access"
```

One is:

```text
HR Policy
```

and another is:

```text
Network Security Policy
```

The question:

> How many days can employees work remotely?

should probably retrieve the HR policy.

Metadata can help constrain the search.

For example:

```text
documentType = "HR_POLICY"
```

Then semantic search operates inside that subset.

---

# 13. Metadata + similarity search

This is a very useful mental model:

```text
User Question
      ↓
Metadata Filter
      ↓
Candidate Documents
      ↓
Similarity Search
      ↓
TopK
      ↓
Threshold
      ↓
Relevant Chunks
```

Not every application needs every step, but this is a common pattern.

---

# 14. What is a Retriever?

So far we've talked about:

```text
Vector Store
```

Now we introduce:

```text
Retriever
```

A **Retriever** is the component responsible for getting relevant documents for a query.

Conceptually:

```text
Question
   ↓
Retriever
   ↓
List<Document>
```

For example:

```java
List<Document> documents =
        retriever.retrieve(
                "How many days can I work remotely?"
        );
```

The retriever might internally perform:

```text
Question
   ↓
Embedding
   ↓
Metadata filtering
   ↓
Vector similarity search
   ↓
TopK
   ↓
Threshold
   ↓
Documents
```

So a retriever is essentially an abstraction around your retrieval strategy.

---

# 15. Vector Store vs Retriever

This distinction is very important for Spring AI.

### Vector Store

Responsible for storing and searching vectors.

```text
VectorStore
     ↓
similarity search
```

### Retriever

Responsible for deciding **how to retrieve relevant documents**.

```text
Retriever
     ↓
VectorStore
     ↓
Search
```

Think:

> **VectorStore = database/search mechanism**

> **Retriever = retrieval strategy**

---

# 16. Spring AI `SearchRequest`

With Spring AI, you can configure a similarity search using a `SearchRequest`.

Conceptually:

```java
List<Document> documents =
        vectorStore.similaritySearch(
                SearchRequest.builder()
                        .query(question)
                        .topK(5)
                        .build()
        );
```

This tells Spring AI:

```text
Query:
question

Return:
up to 5 relevant documents
```

You can also configure similarity constraints and filters depending on the Spring AI version/API you're using.

---

# 17. Metadata filtering in Spring AI

Conceptually, you can specify something like:

```text
department == 'HR'
```

Then your search becomes:

```text
Question:
"What is our remote work policy?"

Filter:
department = HR

topK:
5
```

The idea is:

```text
Vector Store
   │
   ├── Metadata filtering
   │
   └── Similarity search
```

The exact filter expression/API can vary with the Spring AI version, so when you implement this against your current Spring AI version, use the version-specific `SearchRequest` API rather than copying an older example blindly.

---

# 18. Search strategies

Vector similarity search is **not the only retrieval strategy**.

This is where RAG starts becoming more interesting.

## Strategy 1 — Pure vector search

```text
Question
 ↓
Embedding
 ↓
Vector similarity
 ↓
Results
```

Excellent for semantic meaning.

Example:

```text
Question:
"Can I work from home?"

Document:
"Employees may perform their duties remotely..."
```

The words aren't identical, but the meaning is similar.

---

# 19. Keyword search

Traditional search can look for exact terms.

For example:

```text
"OAuth2 PKCE"
```

Keyword search can be very good when exact terms matter.

For example:

```text
Error code:
ERR_TLS_CERTIFICATE
```

A semantic search might understand the general meaning, but exact keyword matching can be valuable.

---

# 20. Hybrid search

Hybrid search combines:

```text
Keyword search
+
Vector search
```

Conceptually:

```text
               Query
                 ↓
        ┌────────┴────────┐
        ↓                 ↓
   Keyword Search    Vector Search
        ↓                 ↓
        └────────┬────────┘
                 ↓
          Combined Ranking
                 ↓
             Results
```

This is often useful for technical documentation.

For example:

```text
"Spring Security OAuth2 AuthorizationServerSettings"
```

Exact keyword matching is valuable, while semantic search helps when the user asks conceptually related questions.

---

# 21. Reranking

Here's another important concept.

Suppose vector search returns:

```text
topK = 20
```

You can then use another model/system to **rerank** those results.

```text
Query
 ↓
Vector Search
 ↓
20 candidate chunks
 ↓
Reranker
 ↓
Top 5 best chunks
```

Why?

Vector similarity gives you a good initial candidate set, but a specialized reranker can sometimes judge:

> "Which of these 20 chunks is actually most relevant to this exact question?"

So:

```text
Retrieval
    ↓
Candidate generation
    ↓
Reranking
    ↓
Final context
```

You don't need to implement reranking yet, but understand where it fits.

---

# 22. Retrieval is not generation

This distinction is critical.

Retriever:

```text
Question
 ↓
Relevant documents
```

LLM:

```text
Question + Relevant documents
 ↓
Answer
```

The retriever doesn't answer:

> "You can work remotely 3 days per week."

It returns:

```text
Document:
Employees can work remotely up to 3 days per week.
```

Then the LLM generates the final response.

---

# 23. What happens when nothing relevant is found?

This is a very important real-world scenario.

User asks:

> What is our policy for working on Mars?

Your HR vector store might have:

```text
Remote work
Leave
Travel
Benefits
Security
```

but nothing about Mars.

A good retrieval system should recognize:

```text
No sufficiently relevant documents
```

rather than blindly passing weak matches to the LLM.

This is one reason similarity thresholds matter.

You want your application to be able to say:

```text
I don't have enough information in the available documents.
```

rather than:

```text
Based on company policy, employees may work on Mars...
```

😂

---

# 24. Retrieval quality vs generation quality

This is another key RAG concept.

Suppose the correct answer exists in your database.

But retrieval returns the wrong chunk.

Then:

```text
Correct information
       ↓
   Vector Store
       ↓
❌ Wrong retrieval
       ↓
       LLM
       ↓
❌ Wrong answer
```

Even if your LLM is excellent.

Therefore:

> **An excellent LLM cannot compensate for consistently bad retrieval.**

This is why RAG engineers pay a lot of attention to retrieval quality.

---

# 25. Precision and recall

You'll eventually encounter these terms when evaluating RAG.

### Precision

Of the chunks you retrieved:

> How many were actually relevant?

Example:

```text
Retrieved = 5
Relevant = 4

Precision = 4/5 = 80%
```

### Recall

Of all the relevant chunks available:

> How many did you successfully retrieve?

Suppose there are 5 relevant chunks in your database and you retrieved 4:

```text
Recall = 4/5 = 80%
```

There is a trade-off.

A tiny `topK` can hurt recall.

A huge `topK` can hurt precision.

That's another reason retrieval configuration matters.

---

# 26. How chunking affects retrieval

This connects directly to your previous topic.

Suppose you created terrible chunks:

```text
Chunk 1:
Employees can work remotely...

Chunk 2:
up to 3 days per week.
```

Question:

```text
How many days can I work remotely?
```

You may retrieve:

```text
Chunk 1
```

but miss:

```text
Chunk 2
```

The retrieval system isn't necessarily broken.

Your **chunking** was poor.

That's why these topics are connected:

```text
Chunking
   ↓
Embedding
   ↓
Retrieval
   ↓
Generation
```

Each stage affects the next.

---

# 27. A practical Spring AI retrieval example

Imagine your vector store contains your HR chunks.

User asks:

```java
String question =
        "How many days can employees work remotely?";
```

You could perform:

```java
List<Document> results =
        vectorStore.similaritySearch(
                SearchRequest.builder()
                        .query(question)
                        .topK(5)
                        .build()
        );
```

Suppose you get:

```text
Document 1:
Employees can work remotely up to 3 days per week.

Document 2:
Remote work requires manager approval.

Document 3:
Employees must remain available during core hours.
```

Then:

```text
results
   ↓
Prompt context
   ↓
ChatClient
   ↓
LLM
```

The LLM receives those documents as context and generates the answer.

---

# 28. A more realistic retrieval pipeline

A production RAG system might eventually look like:

```text
User Question
      ↓
Query preprocessing
      ↓
Metadata filtering
      ↓
Hybrid / vector retrieval
      ↓
topK candidates
      ↓
Similarity threshold
      ↓
Reranking
      ↓
Final relevant chunks
      ↓
Context construction
      ↓
LLM
```

That's much more sophisticated than:

```text
Question → vector search → LLM
```

But **don't try to build all of this immediately**.

Your learning progression should be:

```text
1. Basic similarity search
        ↓
2. topK
        ↓
3. Similarity threshold
        ↓
4. Metadata filtering
        ↓
5. Retriever abstraction
        ↓
6. Hybrid search
        ↓
7. Reranking
```

---

# 29. What I recommend you implement now

For your existing **HR policy + Tika + TokenTextSplitter + pgvector** project, start with:

```text
Question
   ↓
Vector similarity search
   ↓
topK = 3–5
   ↓
Inspect retrieved chunks
```

Don't immediately jump into reranking or hybrid search.

Actually **log the retrieved chunks**.

For example:

```java
for (Document document : results) {
    System.out.println("Content: " + document.getText());
    System.out.println("Metadata: " + document.getMetadata());
}
```

Then ask questions such as:

```text
"What is the remote work policy?"

"How much annual leave do employees get?"

"Who approves remote work?"

"What are the core working hours?"
```

and inspect:

> **Did retrieval actually return the right chunks?**

That exercise will teach you more about RAG than immediately adding another framework abstraction.

---

# 30. The mental model to remember

Your RAG system now looks like:

```text
                    KNOWLEDGE BASE
                         │
                  Documents
                         ↓
                     Chunking
                         ↓
                    Embeddings
                         ↓
                  Vector Store
                         │
                         │
                         ↓
User Question ───► RETRIEVAL
                         │
              ┌──────────┼──────────┐
              ↓          ↓          ↓
            topK     Threshold   Metadata
                                  filter
              └──────────┬──────────┘
                         ↓
                 Relevant Chunks
                         ↓
                  Prompt + Context
                         ↓
                        LLM
                         ↓
                      Answer
```

### The five things to remember

**`topK`**

> Maximum number of results to retrieve.

**Similarity threshold**

> Minimum relevance required for a result to be accepted.

**Metadata filtering**

> Restrict the search to documents matching known attributes.

**Retriever**

> Abstraction that performs the retrieval process and returns relevant `Document`s.

**Search strategy**

> The method used to find/rank relevant information: vector, keyword, hybrid, reranked, etc.

And the most important RAG principle from this topic:

> **Before asking whether the LLM gave a good answer, first ask whether you retrieved the right context.**

If the right context isn't retrieved, the LLM is basically being asked to solve a puzzle while half the pieces are missing.

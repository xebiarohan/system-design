# 23. Vector Databases

## 1. First: Why do we need a Vector Database?

You already learned embeddings:

```text
Text
  ↓
Embedding Model
  ↓
[0.12, -0.83, 0.42, 0.91, ...]
```

An embedding converts text into a **vector of numbers** that represents its semantic meaning.

For example:

```text
"How do I reset my password?"
        ↓
[0.12, -0.83, 0.42, ...]
```

And:

```text
"I forgot my password. How can I change it?"
        ↓
[0.11, -0.80, 0.44, ...]
```

These two vectors will be relatively close because their meanings are similar.

Now imagine you have **100,000 documents**.

You want to answer:

> "How can I reset my password?"

You could theoretically generate an embedding for the question and compare it against every document embedding.

But doing that naively becomes expensive and slow.

That's where a **vector database** comes in.

---

# 2. What is a Vector Database?

A vector database is a database optimized for storing and searching **vectors/embeddings**.

Conceptually, instead of storing only:

```text
id
name
description
```

you store:

```text
id
content
embedding
metadata
```

For example:

| ID | Content                     | Embedding            | Metadata        |
| -- | --------------------------- | -------------------- | --------------- |
| 1  | Password reset instructions | `[0.12, -0.83, ...]` | `department=IT` |
| 2  | Vacation policy             | `[0.71, 0.22, ...]`  | `department=HR` |
| 3  | Account recovery guide      | `[0.14, -0.79, ...]` | `department=IT` |

The important new column is:

```text
embedding
```

---

# 3. Vector Store

Your roadmap uses the term **Vector Store**.

A Vector Store is essentially the abstraction/storage system that lets your application:

```text
store vectors
      +
search vectors
      +
store associated content/metadata
```

Think of it as:

```text
                 Vector Store
              ┌───────────────┐
              │               │
Documents ───→│  Embeddings   │
              │               │
Metadata ────→│  Metadata     │
              │               │
              │  Search       │
              └───────────────┘
```

In Spring AI, you'll later interact with this through the `VectorStore` abstraction.

The nice thing is that your application doesn't need to tightly couple itself to one particular vector database.

For example:

```text
Spring AI
   ↓
VectorStore
   ↓
PostgreSQL + pgvector
```

or:

```text
Spring AI
   ↓
VectorStore
   ↓
Redis
```

or:

```text
Spring AI
   ↓
VectorStore
   ↓
Pinecone
```

Your roadmap recommends starting with **PostgreSQL + pgvector**, rather than trying to master multiple vector databases. 

---

# 4. What exactly is stored?

A vector database usually stores something conceptually like this:

```text
Document
├── id
├── content
├── embedding
└── metadata
```

For example:

```json
{
  "id": "doc-123",
  "content": "Employees can request vacation through the HR portal.",
  "embedding": [0.21, -0.72, 0.34, ...],
  "metadata": {
    "department": "HR",
    "country": "UAE",
    "documentType": "policy"
  }
}
```

There are **three important things** here:

### Content

The actual text:

```text
Employees can request vacation through the HR portal.
```

### Embedding

The numerical representation:

```text
[0.21, -0.72, 0.34, ...]
```

### Metadata

Additional information:

```text
department = HR
country = UAE
documentType = policy
```

Metadata becomes particularly important when you start building production RAG systems.

---

# 5. Similarity Search

This is the heart of vector databases.

Suppose your database contains:

```text
Document A → "How to reset your password"
Document B → "Company vacation policy"
Document C → "How to recover a locked account"
Document D → "How to configure your laptop"
```

User asks:

> "I forgot my password. How can I change it?"

First:

```text
User question
      ↓
Embedding Model
      ↓
Query Vector
```

For example:

```text
[0.13, -0.81, 0.43, ...]
```

The vector database then searches for vectors that are **closest** to this query vector.

Conceptually:

```text
                    Query
                      ●
                     / \
                    /   \
                   /     \
              ●           ●
        Password       Account
         reset         recovery

                               ●
                          Laptop config

                                          ●
                                    Vacation policy
```

The password-reset and account-recovery documents are semantically closer.

So the database returns them.

---

# 6. How does "similar" actually work?

You already studied this in Topic 21.

Similarity can be measured using things like:

* Cosine similarity
* Euclidean distance
* Dot product

For embeddings, **cosine similarity** is very common.

Conceptually:

```text
Similarity(query, document)
             ↓
          score
```

For example:

```text
Document A → 0.94
Document B → 0.82
Document C → 0.31
Document D → 0.12
```

Higher score means more similar, assuming cosine similarity.

So the vector database can rank the documents.

---

# 7. Top-K

This is another key concept from your roadmap.

Suppose there are 100,000 documents.

You don't want:

```text
100,000 documents
```

returned to the LLM.

You might ask for:

```text
Top 5
```

The database returns the five most similar documents.

For example:

```text
Query
 ↓
Vector Database
 ↓
Similarity Search
 ↓
Top 5 documents
```

If the results are:

```text
1. Password reset guide      0.95
2. Account recovery guide    0.91
3. Login troubleshooting     0.87
4. Security FAQ              0.82
5. User account guide        0.79
```

Then:

```text
Top-K = 5
```

means:

> Return the 5 most relevant results.

---

# 8. Why not just use SQL?

This is an important distinction.

Traditional SQL search might do:

```sql
SELECT *
FROM documents
WHERE content LIKE '%password%';
```

This is **keyword-based**.

Suppose the document says:

> "Users can regain access to their account using the recovery procedure."

The user asks:

> "How do I reset my password?"

There might not even be the exact word `"password"` in the document.

A semantic/vector search can still recognize that the concepts are related.

So:

### Traditional search

```text
Keywords
   ↓
Exact/lexical matching
```

### Vector search

```text
Meaning
   ↓
Embedding
   ↓
Semantic similarity
```

That's why vector search is so useful for RAG.

---

# 9. Vector Search vs Keyword Search

Think of Google-like keyword search versus semantic understanding.

### Keyword search

User:

> "car repair"

Document:

> "automobile maintenance"

Keyword matching may struggle because:

```text
car ≠ automobile
repair ≠ maintenance
```

### Semantic search

Embeddings can capture:

```text
car
automobile
vehicle
```

as semantically related concepts.

So:

```text
"car repair"
```

can retrieve:

```text
"automobile maintenance guide"
```

even though the exact words aren't the same.

---

# 10. Metadata

Now we reach the fourth item explicitly listed in your roadmap.

Suppose your company has documents from:

```text
HR
IT
Finance
Legal
```

You ask:

> "What is the password reset procedure?"

There may be 500 documents containing information about accounts.

You might want:

```text
department = IT
```

as an additional constraint.

So instead of simply:

```text
Similarity Search
```

you perform:

```text
Similarity Search
        +
Metadata Filter
```

For example:

```text
Query
 ↓
Embedding
 ↓
Vector Search
 ↓
Filter: department = IT
 ↓
Top-K
```

This is exactly the concept you'll explore more in **Topic 26 — Metadata filtering**. 

---

# 11. Putting everything together

Let's imagine your company's documentation system.

You have:

```text
10,000 documents
```

Each document is split into chunks.

Each chunk gets embedded:

```text
Document chunk
      ↓
Embedding Model
      ↓
Vector
```

Then stored:

```text
┌─────────────────────────────────────┐
│ Vector Database                     │
├─────────────────────────────────────┤
│ content                             │
│ embedding                           │
│ metadata                            │
└─────────────────────────────────────┘
```

Now a user asks:

> "How do I reset my company laptop password?"

Your application does:

```text
User Question
      ↓
Embedding Model
      ↓
Query Vector
      ↓
Vector Database
      ↓
Similarity Search
      ↓
Metadata Filter
      ↓
Top-K
      ↓
Relevant Documents
```

And **this is the retrieval part of RAG**.

The next phase of your roadmap will take those retrieved chunks and give them to the LLM as context. 

---

# 12. The complete RAG picture

This is the architecture you should keep in your head:

```text
              INGESTION
                  │
                  ▼
              Documents
                  │
                  ▼
               Chunking
                  │
                  ▼
            Embedding Model
                  │
                  ▼
        ┌────────────────────┐
        │   Vector Database  │
        │                    │
        │ content            │
        │ embedding          │
        │ metadata           │
        └────────────────────┘
                  ▲
                  │
                  │
              RETRIEVAL
                  │
User Question ────┘
      │
      ▼
Embedding Model
      │
      ▼
Query Vector
      │
      ▼
Similarity Search
      │
      ▼
Metadata Filtering
      │
      ▼
Top-K Results
      │
      ▼
Relevant Context
      │
      ▼
     LLM
      │
      ▼
    Answer
```

That's basically the **core engine behind a simple RAG application**.

---

# 13. What makes a vector database different?

A normal relational database is primarily designed around:

```text
Rows
Columns
Relationships
Transactions
SQL queries
```

A vector database additionally needs to efficiently answer:

> "Which stored vectors are closest to this vector?"

That's a very different type of query.

With millions of vectors, comparing the query against every single vector would be expensive.

Therefore vector databases use specialized **vector indexes / approximate nearest-neighbor (ANN) algorithms** to make similarity searches efficient.

You don't need to go deep into ANN algorithms yet. At Topic 23, understand the conceptual requirement:

```text
Millions of vectors
        ↓
Efficient nearest-neighbor search
        ↓
Top-K similar vectors
```

You'll encounter the implementation details when working with pgvector and other vector stores.

---

# 14. A practical example

Imagine you are building:

**"Chat with company documentation."**

You have:

```text
employee-handbook.pdf
security-policy.pdf
it-guide.pdf
expense-policy.pdf
```

After processing:

```text
PDF
 ↓
Extract text
 ↓
Split into chunks
 ↓
Generate embeddings
 ↓
Store in vector DB
```

The database might contain:

```text
ID   Content                       Metadata
------------------------------------------------
1    Password reset instructions  type=IT
2    VPN configuration             type=IT
3    Annual leave policy           type=HR
4    Expense reimbursement        type=Finance
5    Security incident procedure  type=Security
```

User asks:

> "How can I connect to the company VPN?"

The question becomes a vector:

```text
[0.21, -0.73, 0.55, ...]
```

Vector search might produce:

```text
VPN configuration             0.94
Remote access guide           0.89
Network troubleshooting       0.84
Password policy               0.51
Annual leave policy            0.08
```

With:

```text
Top-K = 3
```

you retrieve:

```text
VPN configuration
Remote access guide
Network troubleshooting
```

Those become the context for the LLM.

---

# 15. The four things you should remember

For **Topic 23**, I'd memorize this mental model:

### ① Vector Store

Where your embeddings and associated data are stored.

```text
Documents → Embeddings → Vector Store
```

### ② Similarity Search

Find vectors whose meanings are closest to the query.

```text
Query Vector
     ↓
Similarity Search
     ↓
Most similar documents
```

### ③ Metadata

Additional information attached to each document/chunk.

```text
department = IT
country = UAE
documentType = policy
```

Useful for filtering.

### ④ Top-K

How many results you want.

```text
Top-K = 5
       ↓
Return 5 most relevant chunks
```

---

# 16. The most important distinction

Don't think:

> **"Vector database = database containing vectors."**

That's technically true, but not enough.

Think:

> **"A vector database allows me to efficiently find semantically similar pieces of information."**

That's the reason it exists.

And the whole flow is:

```text
                 Embedding
                    ↓
Question ───────→ Vector
                    ↓
              Vector Database
                    ↓
             Similarity Search
                    ↓
                  Top-K
                    ↓
             Relevant Context
                    ↓
                   LLM
```

Once this feels natural, **RAG becomes much easier to understand**, because RAG is essentially going to build on this retrieval mechanism. Your roadmap deliberately puts Vector DB immediately before RAG for exactly that reason. 

### One-line summary

> **Vector DB = store embeddings + efficiently search them by semantic similarity + optionally filter by metadata + return Top-K relevant results.**

For your next topic, **Topic 24 — Spring AI `VectorStore`**, the important thing will be seeing how Spring AI turns all of this into Java APIs rather than implementing vector-search infrastructure yourself.

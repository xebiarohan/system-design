# Similarity

## 1. The basic idea

Suppose we have two sentences:

```text
A = "How do I reset my password?"

B = "I forgot my password. How can I change it?"
```

We convert both into embeddings:

```text
A
 ↓
Embedding Model
 ↓
Vector A

B
 ↓
Embedding Model
 ↓
Vector B
```

For simplicity, imagine:

```text
Vector A = [0.8, 0.6, 0.2]

Vector B = [0.7, 0.65, 0.25]
```

Now we need some mathematical method to answer:

```text
How similar is Vector A to Vector B?
```

That's **similarity measurement**.

---

# 2. Why vectors make similarity possible

Remember the mental model from embeddings:

> An embedding represents text as a point in a high-dimensional space.

Imagine we only had 2 dimensions:

```text
Y
│
│       B ●
│      /
│     /
│    ● A
│
└──────────────── X
```

If two points are close or point in similar directions, their underlying text may be semantically similar.

For example:

```text
"How do I reset my password?"
             ● A

"I forgot my password."
             ● B
```

might be close.

Whereas:

```text
"How do I bake a cake?"
             ● C
```

could be far away.

The challenge is:

> How do we mathematically calculate "close" or "similar"?

There are several approaches.

---

# 3. Two important concepts

You should distinguish:

### Similarity

Measures how similar two vectors are.

```text
A ↔ B
```

Higher similarity generally means:

```text
more similar
```

### Distance

Measures how far two vectors are from each other.

```text
A ←──── distance ────→ B
```

Smaller distance generally means:

```text
more similar
```

So they are often conceptually opposite:

```text
Similarity ↑
    ↓
More similar


Distance ↓
    ↓
More similar
```

---

# 4. Vector dimensions

Before looking at similarity algorithms, let's make sure the idea of **dimensions** is clear.

Suppose:

```text
A = [0.2, 0.5, 0.8]
```

This vector has:

```text
3 dimensions
```

You can visualize it as:

```text
A = (x, y, z)

x = 0.2
y = 0.5
z = 0.8
```

For real embedding models:

```text
A = [
    0.123,
    -0.832,
    0.431,
    0.721,
    ...
]
```

There could be hundreds or thousands of dimensions.

For example:

```text
Vector
 ↓
[1536 numbers]
 ↓
1536-dimensional vector
```

---

# 5. Why dimensions matter

Suppose you have:

```text
A = [0.1, 0.2, 0.3]
B = [0.2, 0.3, 0.4]
```

Both have 3 dimensions.

You can compare them.

But:

```text
A = [0.1, 0.2, 0.3]

B = [0.2, 0.3]
```

have different dimensions.

You cannot directly calculate the normal vector similarity between them.

Therefore:

> **Vectors being compared must have compatible dimensions.**

This becomes important when you configure vector databases later.

---

# 6. Distance

Let's start with the easiest concept.

Imagine:

```text
A ●────────● B
```

Distance is simply:

> How far apart are these two points?

For a 2D example:

```text
A = (1, 2)

B = (4, 6)
```

The distance can be calculated using the familiar Pythagorean theorem:

```text
distance = √((x₂-x₁)² + (y₂-y₁)²)
```

So:

```text
distance
= √((4-1)² + (6-2)²)

= √(3² + 4²)

= √25

= 5
```

Easy enough.

But embeddings usually have hundreds/thousands of dimensions.

So we need a generalized formula.

---

# 7. Euclidean distance

For vectors:

```text
A = [a₁, a₂, a₃, ..., aₙ]

B = [b₁, b₂, b₃, ..., bₙ]
```

Euclidean distance is:

```text
d(A,B) =
√(
(a₁-b₁)² +
(a₂-b₂)² +
...
(aₙ-bₙ)²
)
```

In other words:

```text
difference
 ↓
square
 ↓
sum
 ↓
square root
```

For example:

```text
A = [1, 2, 3]

B = [2, 4, 3]
```

Then:

```text
distance
=
√(
(1-2)² +
(2-4)² +
(3-3)²
)

=
√(
1 + 4 + 0
)

=
√5
```

Approximately:

```text
2.236
```

---

# 8. But Euclidean distance isn't always what we want

Here's an interesting problem.

Consider:

```text
A = [1, 1]

B = [10, 10]
```

The vectors point in exactly the same direction:

```text
      B ●
       /
      /
     /
    ● A
```

Their **direction** is identical.

But their Euclidean distance is large.

This matters because embedding similarity often cares more about the **direction/orientation** of vectors than their raw magnitude.

That's where **cosine similarity** becomes extremely useful.

---

# 9. Cosine similarity

Cosine similarity measures:

> **The cosine of the angle between two vectors.**

Imagine:

```text
       B
      /
     /
    / θ
   /
  A
```

Cosine similarity looks at:

```text
angle between A and B
```

rather than simply asking:

```text
how far apart are A and B?
```

The formula is:

```text
cosine_similarity(A, B)
=
(A · B)
----------------
||A|| × ||B||
```

Where:

```text
A · B
```

is the **dot product**.

And:

```text
||A||
```

means the magnitude/length of the vector.

Don't worry—the formula looks scarier than it actually is.

---

# 10. Dot product

Let's calculate one.

Suppose:

```text
A = [1, 2, 3]

B = [4, 5, 6]
```

The dot product is:

```text
A · B

= (1×4)
+ (2×5)
+ (3×6)

= 4 + 10 + 18

= 32
```

So:

```text
A · B = 32
```

---

# 11. Vector magnitude

Magnitude is essentially the length of a vector.

For:

```text
A = [1, 2, 3]
```

the magnitude is:

```text
||A|| = √(1² + 2² + 3²)

      = √14
```

For:

```text
B = [4, 5, 6]
```

we get:

```text
||B|| = √(4² + 5² + 6²)

      = √77
```

---

# 12. Calculate cosine similarity

Now:

```text
cosine_similarity(A,B)
=
(A · B)
----------------
||A|| × ||B||
```

We already have:

```text
A · B = 32

||A|| = √14

||B|| = √77
```

Therefore:

```text
cosine similarity
=
32 / (√14 × √77)
```

Approximately:

```text
0.974
```

That's very high.

So these vectors point in a very similar direction.

---

# 13. Understanding cosine similarity intuitively

Think of vectors as arrows.

### Same direction

```text
A ─────────→

B ───────────────→
```

Angle:

```text
0°
```

Cosine:

```text
cos(0°) = 1
```

So:

```text
similarity = 1
```

Very similar.

---

### Perpendicular

```text
       B
       ↑
       │
       │
       │
       └────────→ A
```

Angle:

```text
90°
```

Cosine:

```text
cos(90°) = 0
```

So:

```text
similarity = 0
```

No directional similarity.

---

### Opposite direction

```text
A ───────→

←──────── B
```

Angle:

```text
180°
```

Cosine:

```text
cos(180°) = -1
```

So:

```text
similarity = -1
```

Opposite direction.

---

# 14. Cosine similarity range

For the mathematical cosine similarity:

```text
-1 ≤ similarity ≤ 1
```

Conceptually:

```text
-1             0              1
│--------------│--------------│
opposite     unrelated      same direction
```

However, **don't assume every embedding model/search system will expose scores in exactly this range or interpret them identically**. Some systems transform or normalize similarity scores.

The important idea is:

```text
higher score
     ↓
more similar
```

when using cosine similarity directly.

---

# 15. Why cosine similarity works well for embeddings

Consider:

```text
A = [1, 1]

B = [10, 10]
```

The vectors have different lengths:

```text
A ───→

B ───────────────→
```

But they point in exactly the same direction.

Cosine similarity:

```text
cos(0°) = 1
```

So:

```text
similarity = 1
```

That's useful because we're primarily interested in the semantic orientation represented by the embedding.

---

# 16. Semantic similarity

Now let's bring this back to language.

Suppose:

```text
A:
"How do I reset my password?"

B:
"I forgot my password and need to change it."
```

Their embeddings might be:

```text
A → [0.31, -0.72, 0.43, ...]

B → [0.29, -0.69, 0.41, ...]
```

Their cosine similarity could be something like:

```text
0.91
```

Now:

```text
C:
"How do I make pizza?"
```

might produce a vector with:

```text
similarity(A,C) = 0.18
```

So:

```text
Query
 │
 ├── Password document → 0.91
 ├── Account document  → 0.72
 ├── Cooking document  → 0.18
 └── Weather document  → 0.11
```

Then we can rank the documents:

```text
0.91  ← most relevant
0.72
0.18
0.11
```

And that's basically the foundation of **vector search**.

---

# 17. The connection to semantic search

Let's say you have 1 million documents.

Each document has an embedding:

```text
Document 1 → Vector 1
Document 2 → Vector 2
Document 3 → Vector 3
...
Document 1M → Vector 1M
```

User asks:

```text
"What is the company's work-from-home policy?"
```

First:

```text
Question
   ↓
Embedding Model
   ↓
Query Vector
```

Then:

```text
Query Vector
      ↓
Compare with document vectors
      ↓
Similarity score
      ↓
Rank documents
```

For example:

```text
Document                         Score

Remote work policy               0.93
Office attendance policy         0.86
Employee benefits                0.64
Security policy                  0.31
Company history                  0.12
```

Take the top few:

```text
Top-K
```

and use them for retrieval.

This connects directly to your roadmap's later **Vector Database → RAG** phases. The roadmap explicitly introduces vector stores, similarity search, metadata, and Top-K after embeddings. 

---

# 18. What is Top-K?

You'll see this constantly in RAG.

Suppose similarity results are:

```text
Document A → 0.94
Document B → 0.91
Document C → 0.87
Document D → 0.52
Document E → 0.31
```

If:

```text
K = 3
```

we select:

```text
A
B
C
```

because they're the top 3 most similar documents.

```text
Query
 ↓
Vector
 ↓
Similarity Search
 ↓
Rank
 ↓
Top-K
 ↓
Relevant documents
```

You'll study this more deeply in the Vector Database phase.

---

# 19. Similarity threshold

Another important concept is a **similarity threshold**.

Suppose:

```text
threshold = 0.75
```

and results are:

```text
A → 0.92
B → 0.87
C → 0.81
D → 0.63
E → 0.42
```

Then:

```text
A ✓
B ✓
C ✓
D ✗
E ✗
```

because:

```text
score >= 0.75
```

This can prevent irrelevant documents from being passed to the LLM.

Later, in RAG, this becomes:

```text
Question
 ↓
Embedding
 ↓
Vector Search
 ↓
Similarity threshold
 ↓
Relevant chunks
 ↓
LLM
```

---

# 20. Similarity ≠ truth

This is a **very important RAG concept**.

Suppose the user asks:

```text
"What is our vacation policy?"
```

The vector search finds:

```text
Document A → similarity 0.94
```

That means:

> The document is semantically similar to the question.

It does **not** mean:

> The document contains the correct answer.

Similarity is about **relevance**, not truth.

For example:

```text
Query:
"How many vacation days do employees get?"

Document:
"Employees can request vacation through the HR portal."
```

This could be semantically related but doesn't actually answer the question.

So:

```text
Similarity
    ↓
Relevant?
```

not:

```text
Similarity
    ↓
Correct?
```

This distinction becomes very important when you study **RAG evaluation** later.

---

# 21. Similarity vs keyword search

Let's compare them.

### Keyword search

Query:

```text
"How can I work from home?"
```

Searches for words such as:

```text
work
home
```

But the document says:

```text
"Employees are permitted to work remotely three days per week."
```

A keyword search may not perform well.

---

### Semantic search

Embedding:

```text
"How can I work from home?"
```

and:

```text
"Employees are permitted to work remotely three days per week."
```

may be close in vector space.

So:

```text
"work from home"
       ≈
"work remotely"
```

from a semantic perspective.

---

# 22. A very useful example

Imagine your company knowledge base contains:

```text
Doc 1:
"Employees can work remotely on Monday, Wednesday and Friday."

Doc 2:
"Office parking is available from 7 AM."

Doc 3:
"Annual leave is 30 days."

Doc 4:
"Employees working remotely must use VPN."
```

User asks:

```text
"What are the rules for working from home?"
```

Embedding search might produce:

```text
Doc 1 → 0.94
Doc 4 → 0.89
Doc 3 → 0.41
Doc 2 → 0.12
```

The top two documents are retrieved.

Then RAG gives those documents to the LLM:

```text
Question
   +
Doc 1
   +
Doc 4
   ↓
LLM
   ↓
Answer
```

This is why embeddings are so important.

---

# 23. Cosine similarity vs Euclidean distance

You should know the difference conceptually.

|                      | Cosine Similarity | Euclidean Distance    |
| -------------------- | ----------------- | --------------------- |
| Measures             | Angle/direction   | Physical distance     |
| Higher means         | More similar      | Less similar          |
| Lower means          | Less similar      | More similar          |
| Range                | Usually -1 to 1   | 0 to ∞                |
| Focus                | Direction         | Magnitude + direction |
| Common in embeddings | Very common       | Also used             |

Think:

```text
Cosine:
"Are these arrows pointing in the same direction?"

Euclidean:
"How far apart are these points?"
```

---

# 24. A subtle but important point: normalization

Sometimes vectors are **normalized**.

Normalization means converting a vector so its magnitude becomes:

```text
1
```

For example:

```text
A = [3, 4]
```

Magnitude:

```text
√(3² + 4²)
= 5
```

Normalized vector:

```text
A' = [3/5, 4/5]

   = [0.6, 0.8]
```

Now:

```text
||A'|| = 1
```

Why is this useful?

Because if vectors are normalized, cosine similarity becomes closely related to the dot product.

```text
cosine similarity
       ≈
dot product
```

This can make vector calculations more efficient.

You don't need to implement this yourself for basic Spring AI usage, but you should understand the concept.

---

# 25. Why vector databases exist

Now you can see a problem.

Suppose you have:

```text
10 million documents
```

Each document has an embedding.

For every user query, theoretically you could:

```text
Query
 ↓
Embedding
 ↓
Compare against 10 million vectors
 ↓
Sort
 ↓
Top-K
```

That's expensive.

This is why specialized **vector databases/vector indexes** exist.

They use algorithms and indexes designed to efficiently find approximate or exact nearest neighbors.

The overall process becomes:

```text
Query Vector
      ↓
Vector Database
      ↓
Nearest-neighbor search
      ↓
Top-K vectors
```

You'll get into this properly in Topics 23–25.

---

# 26. Nearest neighbor

You will hear this phrase frequently:

> **Nearest Neighbor Search**

Imagine:

```text
                A ●
                  \
                   \
                    ● Query
                   /
                  /
             B ●
```

The database wants to find:

```text
Which vectors are closest to the query?
```

These are the **nearest neighbors**.

With embeddings:

```text
Query
 ↓
Nearest vectors
 ↓
Semantically similar documents
```

This is the mathematical foundation behind vector retrieval.

---

# 27. One subtle misconception to avoid

Don't think:

> "The embedding model knows that two sentences are 90% similar."

It doesn't necessarily output:

```text
90%
```

The embedding model outputs vectors.

Then **your similarity algorithm** calculates a score.

So the architecture is:

```text
Text A ──→ Embedding Model ──→ Vector A
                                  │
                                  │
                                  ↓
                              Similarity
                                  ↑
                                  │
Text B ──→ Embedding Model ──→ Vector B
```

The embedding model creates the representation.

The similarity algorithm compares the representations.

These are two separate responsibilities.

---

# 28. Spring AI perspective

Eventually you'll use Spring AI's `EmbeddingModel`.

Conceptually:

```java
EmbeddingModel embeddingModel;
```

You provide text:

```java
String text = "How do I reset my password?";
```

and obtain an embedding.

Then you can compare embeddings using a similarity function/library, or more commonly let a vector store perform the similarity search for you.

The architecture becomes:

```text
Spring Boot
     │
     ↓
EmbeddingModel
     │
     ↓
Vector
     │
     ↓
VectorStore
     │
     ↓
Similarity Search
     │
     ↓
Top-K Documents
```

That leads naturally to your next topics around vector databases and `VectorStore`. 

---

# 29. The complete RAG picture

Now combine Topics 20 and 21.

### During ingestion

```text
PDF
 ↓
Extract text
 ↓
Chunks
 ↓
Embedding Model
 ↓
Vectors
 ↓
Vector Database
```

### During query

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
Top-K Chunks
      ↓
Prompt + Context
      ↓
LLM
      ↓
Answer
```

This is the core RAG pipeline you'll eventually build.

---

# 30. The four things you should remember

Your roadmap lists four things for Topic 21, so make sure you can explain each one.

### 1. Vector dimensions

```text
[0.2, 0.5, 0.8]
```

has:

```text
3 dimensions
```

Real embeddings can have hundreds or thousands of dimensions.

---

### 2. Distance

Measures:

```text
How far apart are two vectors?
```

Common example:

```text
Euclidean distance
```

Smaller distance generally means greater similarity.

---

### 3. Cosine similarity

Measures:

```text
How similar are the directions of two vectors?
```

Formula:

```text
cos(A,B) =
(A · B) / (||A|| × ||B||)
```

For direct cosine similarity:

```text
1   → same direction
0   → perpendicular
-1  → opposite direction
```

---

### 4. Semantic similarity

Means:

```text
Text meaning
     ↓
Embedding
     ↓
Vector relationship
     ↓
Similarity
```

For example:

```text
"How do I reset my password?"

        ≈

"I forgot my password. How can I change it?"
```

even though the exact words aren't identical.

---

# 31. Final mental model

This is the diagram I'd keep in your notes:

```text
                  EMBEDDING + SIMILARITY

Text A                              Text B
  │                                    │
  ↓                                    ↓
Embedding Model                  Embedding Model
  │                                    │
  ↓                                    ↓
Vector A                            Vector B
  │                                    │
  └──────────────┬─────────────────────┘
                 ↓
           Similarity Measure
                 │
        ┌────────┴────────┐
        ↓                 ↓
   Cosine Similarity   Distance
        │                 │
        └────────┬────────┘
                 ↓
          Similarity Score
                 │
                 ↓
         Rank / Top-K results
```

And the bigger picture:

```text
                    User Question
                          │
                          ↓
                   Embedding Model
                          │
                          ↓
                     Query Vector
                          │
                          ↓
                  Vector Database
                          │
                    Similarity Search
                          │
                          ↓
                       Top-K
                          │
                          ↓
                   Relevant Chunks
                          │
                          ↓
                    ┌───────────┐
                    │    LLM    │
                    └─────┬─────┘
                          │
                          ↓
                       Answer
```

### 🔑 The key distinction

The most important thing to understand before moving on is:

> **Embedding converts meaning into a vector. Similarity measures the relationship between vectors.**

```text
Embedding = representation
Similarity = comparison
Vector DB = efficient retrieval
RAG = retrieval + LLM generation
```

Once that clicks, **Topic 22 — Spring AI `EmbeddingModel`** becomes much more practical: instead of treating it as another Spring API, you'll understand exactly **what problem that API is solving**.

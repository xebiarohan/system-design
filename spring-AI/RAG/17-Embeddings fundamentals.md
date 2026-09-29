# Phase 7 — Embeddings

Your roadmap defines **Topic 20 — Embedding fundamentals** as a 2-hour topic. The core idea is to understand how text is converted into vectors:

```text
Text
 ↓
Embedding Model
 ↓
[0.12, -0.83, 0.42, ...]
```



Since you're a Java/Spring developer, I'll explain this from both the **AI perspective** and the **software-architecture perspective**. This topic is especially important because embeddings are the foundation for the RAG phase you'll reach later.

---

# 1. What is an embedding?

An **embedding** is a numerical representation of some data—usually text—where the numbers capture **semantic meaning**.

For example:

```text
"Java is a programming language"
```

might be converted into something conceptually like:

```text
[0.21, -0.43, 0.87, 0.15, -0.92, ...]
```

This array is called a **vector**.

So:

```text
Text
 ↓
Embedding Model
 ↓
Vector
```

The key idea is:

> **Similar meanings should produce vectors that are close to each other in vector space.**

That's the entire foundation behind semantic search.

---

# 2. Why do we need embeddings?

Imagine you have this database:

```text
1. "Java is an object-oriented programming language."
2. "Spring Boot simplifies Java application development."
3. "The weather in Dubai is hot today."
4. "React is a JavaScript UI library."
```

Now the user searches:

```text
"What is Java used for?"
```

A traditional keyword search might look for:

```text
Java
```

and return documents containing the exact word.

But embeddings allow something more powerful.

The query:

```text
"What is Java used for?"
```

can be converted into a vector:

```text
Query
 ↓
Embedding Model
 ↓
[0.31, -0.51, 0.76, ...]
```

And each document has already been converted:

```text
Document 1 → [0.29, -0.48, 0.73, ...]
Document 2 → [0.25, -0.41, 0.70, ...]
Document 3 → [-0.82, 0.72, -0.15, ...]
Document 4 → [0.12, -0.15, 0.31, ...]
```

Now we can calculate which vectors are closest.

The result might be:

```text
Query
  │
  ├── Document 1  ← very similar
  ├── Document 2  ← similar
  ├── Document 4  ← somewhat similar
  └── Document 3  ← not similar
```

Notice something important:

**The system doesn't need the exact words to match.**

It is looking for **meaning**.

---

# 3. Embedding vs normal text

Let's take two sentences:

```text
A: "How do I create a Java application?"
B: "What are the steps to build a Java program?"
```

They use different words:

```text
create
application

vs

steps
build
program
```

But semantically they're very similar.

An embedding model tries to represent that similarity.

Conceptually:

```text
A
 ↓
[0.81, 0.21, -0.43, 0.67, ...]

B
 ↓
[0.79, 0.24, -0.41, 0.65, ...]
```

The vectors are close.

Compare:

```text
C: "What is the weather in Dubai?"
```

Its vector might be much farther away:

```text
A ─────── B


             C
```

That's the magic of embeddings.

---

# 4. What exactly is a vector?

As a Java developer, think of a vector as:

```java
float[] vector;
```

For example:

```java
float[] vector = {
    0.12f,
    -0.83f,
    0.42f,
    0.71f,
    -0.19f
};
```

In reality, an embedding might contain hundreds or thousands of dimensions.

For example:

```text
[0.12, -0.83, 0.42, 0.71, -0.19, ...]
                         ↑
                   many dimensions
```

You don't normally interpret each number individually.

This is important.

You cannot look at:

```text
0.12
```

and say:

> "This number represents Java."

And:

```text
-0.83
```

means:

> "This number represents programming."

It doesn't work that way.

**Meaning is distributed across the vector dimensions.**

---

# 5. What is a dimension?

Suppose an embedding model generates:

```text
[0.2, 0.7, -0.4]
```

This vector has:

```text
3 dimensions
```

A real model might produce something like:

```text
[0.12, -0.83, 0.42, ...]
```

with hundreds or thousands of dimensions.

So:

```text
Embedding dimension
=
number of values in the vector
```

For example:

```text
Embedding:

[0.1, 0.2, 0.3]

dimension = 3
```

If:

```text
[0.1, 0.2, 0.3, 0.4, ...]
```

contains 1536 values:

```text
dimension = 1536
```

The exact dimensionality depends on the embedding model.

---

# 6. Why so many dimensions?

Language is extremely complex.

Consider:

```text
"Apple"
```

It could mean:

```text
🍎 fruit
```

or:

```text
Apple the company
```

And:

```text
"Java"
```

could refer to:

```text
Java programming language
```

or:

```text
Java island
```

Embeddings encode many aspects of context and meaning into a high-dimensional representation.

You can think of it as creating a point in a very large mathematical space:

```text
                 Vector Space

                     • Java
                   /
                  /
       Spring •
                \
                 \
                  • Python
```

The actual space may have 768, 1024, 1536, 3072, or other dimensions depending on the model.

Humans obviously can't visualize that directly.

---

# 7. The most important property: semantic similarity

This is the key thing to understand.

Consider:

```text
Sentence 1:
"How can I reset my password?"

Sentence 2:
"I forgot my password. How do I change it?"
```

Different words.

Similar meaning.

Therefore:

```text
Embedding 1
      ↓
   [....]

Embedding 2
      ↓
   [....]
```

should be relatively close.

Now:

```text
Sentence 3:
"How do I make chocolate cake?"
```

has very different meaning.

Its embedding should be farther away.

So:

```text
             password
                ●
              /   \
             /     \
            ●       ●
       password   password


                         ●
                     chocolate
                        cake
```

This gives us the ability to search by **meaning** instead of exact keywords.

---

# 8. Embedding model

The component responsible for converting text into vectors is an:

**Embedding Model**

Architecture:

```text
             Text
              │
              ↓
       ┌───────────────┐
       │   Embedding   │
       │     Model     │
       └───────┬───────┘
               │
               ↓
        Vector / Embedding
```

For example:

```text
"Spring Boot is great"
```

goes into the model.

Output:

```text
[0.14, -0.82, 0.33, 0.71, ...]
```

That vector is the embedding.

---

# 9. Embedding model vs Chat Model

This is extremely important because you've already learned about chat models.

They're different things.

### Chat model

Input:

```text
Prompt
```

Output:

```text
Text
```

```text
Prompt
 ↓
Chat Model
 ↓
Text
```

For example:

```text
"Explain OAuth 2.0"
```

↓

```text
"OAuth 2.0 is an authorization framework..."
```

---

### Embedding model

Input:

```text
Text
```

Output:

```text
Vector
```

```text
Text
 ↓
Embedding Model
 ↓
Vector
```

For example:

```text
"Explain OAuth 2.0"
```

↓

```text
[0.21, -0.83, 0.44, ...]
```

So:

| Model           | Input       | Output     |
| --------------- | ----------- | ---------- |
| Chat Model      | Text/prompt | Text       |
| Embedding Model | Text        | Vector     |
| Image Model     | Image/text  | Image      |
| Speech Model    | Audio/text  | Audio/text |

---

# 10. Embeddings don't generate answers

This is another common misconception.

Suppose you send:

```text
"What is Spring Boot?"
```

to an embedding model.

It doesn't respond:

```text
"Spring Boot is a framework..."
```

Instead:

```text
"What is Spring Boot?"
       ↓
Embedding Model
       ↓
[0.21, -0.52, 0.73, ...]
```

The embedding model is **not answering your question**.

It's converting the question into a mathematical representation.

---

# 11. How embeddings are used

The most important applications are:

### Semantic search

```text
Query
 ↓
Embedding
 ↓
Find similar vectors
 ↓
Relevant documents
```

### RAG

```text
User question
 ↓
Embedding
 ↓
Vector search
 ↓
Relevant documents
 ↓
LLM
 ↓
Answer
```

### Recommendation systems

```text
Product
 ↓
Embedding
 ↓
Find similar products
```

### Document clustering

```text
Documents
 ↓
Embeddings
 ↓
Group similar documents
```

### Duplicate detection

```text
Document A
 ↓
Embedding

Document B
 ↓
Embedding

Compare
 ↓
Similarity
```

Your roadmap is ultimately taking you toward **RAG**, where embeddings become one of the core building blocks. The roadmap describes RAG as retrieving relevant information and adding that context to the prompt before calling the LLM. 

---

# 12. Embeddings and RAG

This is probably the most important connection for your roadmap.

Suppose you have:

```text
company-policies.pdf
```

containing:

```text
Employees can work remotely up to 3 days per week.
```

You want your AI application to answer:

```text
"How many days can I work remotely?"
```

First, you process the document.

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

Later:

```text
User question
 ↓
Embedding Model
 ↓
Query vector
 ↓
Vector Database
 ↓
Find similar vectors
 ↓
Relevant chunk
 ↓
LLM
 ↓
Answer
```

So embeddings are effectively the **bridge between human language and vector search**.

---

# 13. Why not just use a database search?

Suppose your database contains:

```text
"Employees may work remotely three days per week."
```

User asks:

```text
"How many days can I work from home?"
```

A simple SQL:

```sql
WHERE text LIKE '%work from home%'
```

might fail because the document says:

```text
work remotely
```

instead of:

```text
work from home
```

Embeddings solve this semantic mismatch.

```text
"work from home"
       ↓
    vector A

"work remotely"
       ↓
    vector B

distance(A, B)
       ↓
    small
```

Therefore the system understands that they're semantically related.

---

# 14. How does the embedding model learn meaning?

This gets into the internals.

At a very high level:

```text
Training text
      ↓
Neural network
      ↓
Learn relationships between words/tokens
      ↓
Learn semantic representations
      ↓
Embedding
```

The model sees huge amounts of language during training.

It learns relationships such as:

```text
king
queen

Java
programming

doctor
hospital

car
vehicle
```

The important thing is that **embeddings are learned representations**.

The model isn't using a manually created dictionary such as:

```text
Java = programming
Spring = framework
car = vehicle
```

Instead, these representations emerge from the model's training.

---

# 15. A useful mental model

Think of an embedding as a **GPS coordinate for meaning**.

Imagine a huge map:

```text
                 Programming
                      ●
                    /   \
                   /     \
             Java ●       ● Python
                 /
                /
         Spring ●


                           ● Weather
                            


     ● Cooking
```

Each piece of text gets placed somewhere on this conceptual map.

Similar meanings:

```text
        Java
         ●
        /
       ● Spring
```

are close.

Unrelated meanings:

```text
Java ●


                         ● Pizza
```

are farther apart.

The actual embedding space is much more complicated and has many dimensions, but this is an excellent mental model.

---

# 16. Similarity is not the same as equality

This distinction matters.

Suppose:

```text
A = "Java programming language"
B = "Java programming"
```

Their embeddings may be close.

But they're not identical.

So embedding search generally asks:

> "How similar are these two vectors?"

rather than:

> "Are these two strings equal?"

This leads directly into your next topic:

**Topic 21 — Similarity**

Your roadmap specifically calls out:

* Cosine similarity
* Distance
* Vector dimensions
* Semantic similarity 

That's where you'll learn how we mathematically determine whether two embeddings are close.

---

# 17. What happens inside the model?

You don't need to understand the complete neural-network mathematics yet, but understand the conceptual pipeline.

For:

```text
"Spring Boot is a Java framework"
```

the process is roughly:

```text
Text
 ↓
Tokenization
 ↓
Tokens
 ↓
Neural network
 ↓
Hidden representations
 ↓
Pooling / representation selection
 ↓
Embedding vector
```

Eventually:

```text
[0.123,
 -0.432,
  0.891,
  0.214,
  ...]
```

The exact architecture depends on the embedding model.

You don't need to memorize the internal implementation of every provider.

---

# 18. Embedding one sentence vs a document

You can embed different types of text:

```text
Word
Sentence
Paragraph
Document chunk
Query
```

For RAG, we usually don't embed a giant document as one huge vector.

Instead:

```text
Large PDF
   ↓
Split into chunks
   ↓
Chunk 1 → embedding
Chunk 2 → embedding
Chunk 3 → embedding
Chunk 4 → embedding
...
```

For example:

```text
PDF
 │
 ├── Chunk 1
 │      ↓
 │   Vector 1
 │
 ├── Chunk 2
 │      ↓
 │   Vector 2
 │
 ├── Chunk 3
 │      ↓
 │   Vector 3
```

This is why your roadmap later spends significant time on **chunking** in the RAG phase. 

---

# 19. Query embedding and document embedding

Here's a critical RAG concept.

Suppose:

```text
Document:
"Employees are entitled to 30 days of annual leave."
```

We generate:

```text
Document
 ↓
Embedding Model
 ↓
Document Vector
```

Then user asks:

```text
"How much vacation can employees take?"
```

We generate:

```text
Question
 ↓
Same / compatible Embedding Model
 ↓
Query Vector
```

Then:

```text
Query Vector
       ↓
Similarity Search
       ↓
Document Vector
       ↓
Relevant document
```

So:

```text
Document → Vector
Question → Vector

       ↓

Compare vectors
```

This is the core of semantic retrieval.

---

# 20. Why should we usually use the same embedding model?

Imagine the document was embedded using:

```text
Embedding Model A
```

but your query is embedded using:

```text
Embedding Model B
```

The vector spaces may not be compatible.

You could have:

```text
Model A:
1536 dimensions

Model B:
768 dimensions
```

You can't meaningfully compare:

```text
[1536 values]
```

against:

```text
[768 values]
```

Even if dimensions happened to match, different models can define different vector spaces.

So typically:

```text
Documents
    ↓
Embedding Model X
    ↓
Vectors

Query
    ↓
Embedding Model X
    ↓
Vector

Compare
```

Same model/compatible embedding space is the safe rule.

---

# 21. Embedding dimension and vector databases

Suppose your model produces:

```text
1536-dimensional vectors
```

Your vector database needs to know:

```text
dimension = 1536
```

For example conceptually:

```text
VectorStore
 ├── dimension = 1536
 ├── vector 1
 ├── vector 2
 ├── vector 3
 └── ...
```

If you later switch to a model producing:

```text
3072 dimensions
```

you generally can't just put those vectors into the same vector field expecting 1536 dimensions.

You need to account for the model change.

This becomes an important production concern later.

---

# 22. Embeddings are not magic "meaning numbers"

One subtle point:

Don't think:

```text
embedding = dictionary of meaning
```

It's better to think:

```text
embedding = learned numerical representation
```

The individual dimensions aren't usually human interpretable.

What matters is the **geometry of the vector space**:

```text
distance
direction
relative position
similarity
```

That's why Topic 21 follows immediately after Topic 20.

---

# 23. Embeddings vs keywords

Here's a very useful comparison.

### Keyword search

```text
"How do I reset my password?"
```

looks for:

```text
reset
password
```

Potential problem:

```text
"I forgot my credentials and need to change them."
```

No exact keyword match.

---

### Semantic search

```text
"How do I reset my password?"
        ↓
      vector
        ↓
Find semantically similar content
        ↓
"I forgot my credentials and need to change them."
```

Much better for natural-language questions.

---

# 24. Where Spring AI fits

Now connect this to Spring AI.

You already learned:

```text
ChatClient
ChatModel
```

For embeddings, Spring AI provides:

```text
EmbeddingModel
```

Conceptually:

```java
EmbeddingModel embeddingModel;
```

Then:

```text
Text
 ↓
EmbeddingModel
 ↓
float[] / List<Double>
```

Spring AI gives you an abstraction so your application doesn't have to directly implement provider-specific embedding API calls.

Your roadmap specifically introduces `EmbeddingModel` in Topic 22 after you've learned the fundamentals and similarity mathematics. 

---

# 25. The complete picture

At this point, remember this:

```text
                    EMBEDDINGS

                       Text
                        │
                        ↓
                ┌──────────────┐
                │  Embedding    │
                │    Model      │
                └──────┬───────┘
                       │
                       ↓
                    Vector
                       │
                       ↓
              ┌─────────────────┐
              │ Vector Database │
              └────────┬────────┘
                       │
                       │ similarity search
                       │
                       ↓
                 Relevant text
```

For RAG:

```text
                     DOCUMENT
                        │
                        ↓
                     Chunking
                        │
                        ↓
                  Embedding Model
                        │
                        ↓
                     Vectors
                        │
                        ↓
                  Vector Database
                        │
                        │
             ┌──────────┘
             │
             │
USER ────────┘
Question
   │
   ↓
Embedding Model
   │
   ↓
Query Vector
   │
   ↓
Similarity Search
   │
   ↓
Relevant Chunks
   │
   ↓
Prompt + Context
   │
   ↓
   LLM
   │
   ↓
 Answer
```

That last diagram is **the road you're ultimately building toward with Phase 7 → Phase 8 → Phase 9**.

---

# 26. What you should know after Topic 20

Before moving to Topic 21, make sure you can answer these questions:

### Basic

**What is an embedding?**

> A numerical vector representation of data, typically text, that captures useful semantic relationships.

**What does an embedding model do?**

```text
Text → Vector
```

**What is a vector?**

> An ordered collection of numerical values representing a piece of data in a multidimensional space.

---

### Intermediate

**Why are embeddings useful?**

Because semantically similar text can have vectors that are close together, enabling semantic search and retrieval.

**Why do embeddings have hundreds/thousands of numbers?**

Because language has many dimensions of information, and the model represents that information in a high-dimensional space.

**Can I interpret each number individually?**

Generally, no. The representation is distributed across the vector.

---

### RAG-level

**Why embed documents?**

```text
Document
 ↓
Vector
 ↓
Store
```

So that we can efficiently find semantically relevant content later.

**Why embed the user query?**

```text
Question
 ↓
Vector
 ↓
Compare against document vectors
```

**Why is the same embedding model normally used for both?**

Because the query and document vectors need to exist in the same compatible vector space.

---

# 27. The one sentence I'd remember

If you remember only one thing from Topic 20, make it this:

> **An embedding converts text into a high-dimensional numerical vector such that semantic relationships between pieces of text can be represented geometrically, allowing us to perform operations like semantic similarity and search.**

And the progression of your roadmap now makes perfect sense:

```text
20. Embedding fundamentals
          ↓
21. Similarity
          ↓
22. Spring AI EmbeddingModel
          ↓
23. Vector databases
          ↓
24. Spring AI VectorStore
          ↓
25. PostgreSQL + pgvector
          ↓
RAG 🚀
```

**Topic 21 — Similarity** is the natural next step. That's where we'll take these vectors and answer the really interesting question: **"How does the computer mathematically decide that these two sentences mean similar things?"**

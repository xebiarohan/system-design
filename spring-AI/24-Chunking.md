# 29. Chunking

Suppose your document contains:

```text
HR Policy

Employees can work remotely up to 3 days per week.

Remote work requires manager approval.

Employees must be available during core working hours.

Annual leave must be requested at least 2 weeks in advance.

...
```

You don't necessarily want to create **one embedding for the entire document**.

Instead:

```text
Document
   ↓
Chunk 1
Chunk 2
Chunk 3
Chunk 4
...
```

Then each chunk gets its own embedding.

---

# 1. Why do we need chunking?

Imagine a 100-page company handbook.

If you embed the entire handbook as one vector:

```text
100-page handbook
       ↓
    1 vector
```

What does that vector represent?

It represents the **overall semantic meaning of the entire handbook**.

Now the user asks:

> "How many days of remote work are allowed?"

You want to retrieve the small section:

```text
Employees can work remotely up to 3 days per week.
```

But your only vector represents:

```text
HR
+
Leave
+
Travel
+
Security
+
Remote work
+
Benefits
+
Payroll
+
...
```

That's not ideal.

Instead:

```text
100-page handbook
       ↓
     500 chunks
       ↓
    500 vectors
```

Now the remote-work chunk can have a vector strongly representing:

```text
remote work
working remotely
3 days
```

So retrieval becomes much more precise.

---

# 2. Chunk size

**Chunk size = how much content you put into one chunk.**

For example:

```text
Chunk size = 500 tokens
```

means each chunk is approximately 500 tokens.

You might have:

```text
Document
 ↓
500 tokens → Chunk 1
500 tokens → Chunk 2
500 tokens → Chunk 3
500 tokens → Chunk 4
```

The exact size depends on the splitter and configuration.

---

# 3. Why not make chunks extremely small?

Suppose you split this:

```text
Employees can work remotely up to 3 days per week.
```

into:

```text
Chunk 1:
Employees

Chunk 2:
can work

Chunk 3:
remotely

Chunk 4:
up to 3 days

Chunk 5:
per week
```

That's obviously terrible.

The individual chunks don't contain enough meaning.

If the user asks:

> "How many days can I work remotely?"

The chunk:

```text
up to 3 days
```

doesn't tell you:

* Who?
* What?
* Under what policy?
* Is this about remote work?

You have lost context.

---

# 4. Why not make chunks extremely large?

Now imagine:

```text
Chunk = 20,000 tokens
```

You may end up with:

```text
Chunk:
HR
+ Leave
+ Security
+ Travel
+ Remote work
+ Benefits
+ Payroll
+ ...
```

Again, the embedding represents too many different concepts.

Retrieval becomes less precise.

There is also another problem:

```text
Retrieved chunk
      ↓
Huge amount of text
      ↓
LLM context
```

You're giving the LLM much more information than it needs.

So there is a trade-off:

```text
Too small
   ↓
Not enough context

Too large
   ↓
Too much unrelated context
```

The goal is:

> **Small enough for precise retrieval, but large enough to preserve meaning.**

---

# 5. Token vs character vs words

Chunk size can be measured differently.

For example:

```text
500 characters
500 words
500 tokens
```

These are **not the same thing**.

For LLM applications, token-based chunking is often useful because LLM context limits are measured in tokens.

You've already used:

```java
TokenTextSplitter
```

in your HR-policy example.

For example, you previously had something conceptually like:

```java
TokenTextSplitter.builder()
        .withChunkSize(...
```

That means your splitting strategy is based on **tokens**, rather than simply counting Java characters.

---

# 6. Chunk overlap

Now we reach an extremely important concept.

Suppose you have:

```text
Chunk 1:
Employees can work remotely up to 3 days per week.
Remote work requires manager approval.
Employees must remain available...
```

and the next chunk starts:

```text
Chunk 2:
during core working hours.
Annual leave must...
```

You could accidentally split information that belongs together.

That's where **overlap** helps.

---

# 7. What is chunk overlap?

Suppose:

```text
Chunk size = 500 tokens
Overlap = 100 tokens
```

Then:

```text
Chunk 1
├──────────────────────────────┤
          500 tokens

              Chunk 2
              ├──────────────────────────────┤
              100 overlap
```

Conceptually:

```text
Chunk 1:
[A B C D E F G H I J]

Chunk 2:
                  [I J K L M N O P Q R]
                  ↑
              overlap
```

`I J` appears in both chunks.

---

# 8. Why is overlap useful?

Consider this sentence:

```text
Employees working remotely must obtain approval
from their manager before beginning remote work.
```

Suppose a bad split happens:

```text
Chunk 1:
Employees working remotely must obtain approval
from their manager

Chunk 2:
before beginning remote work.
```

Chunk 2 doesn't make much sense by itself.

With overlap:

```text
Chunk 1:
Employees working remotely must obtain approval
from their manager before beginning remote work.

Chunk 2:
from their manager before beginning remote work.
...
```

Now both chunks preserve enough context.

---

# 9. Chunk overlap doesn't mean duplicate everything

A common misunderstanding is:

> "More overlap = better RAG."

Not necessarily.

Suppose:

```text
Chunk size = 500
Overlap = 400
```

You are effectively storing a huge amount of repeated information.

That causes:

* More chunks
* More embeddings
* More storage
* More retrieval duplicates
* Higher processing cost

For example:

```text
Chunk 1:
A B C D E F G H I J

Chunk 2:
      G H I J K L M N O P

Chunk 3:
            M N O P Q R S T U V
```

You're embedding a lot of the same information repeatedly.

So overlap should be **large enough to preserve boundary context, but not unnecessarily large**.

---

# 10. Recursive splitting

This is one of the most useful general-purpose approaches.

Instead of blindly cutting text every N characters/tokens, **recursive splitting tries progressively smaller separators**.

Imagine this document:

```text
# Remote Work Policy

## Eligibility

All full-time employees are eligible.

## Remote Days

Employees can work remotely up to 3 days per week.

## Approval

Manager approval is required.
```

You ideally want to preserve:

```text
Section
 ↓
Paragraph
 ↓
Sentence
```

rather than randomly cutting:

```text
"Employees can work remotely up to 3..."
```

---

# 11. How recursive splitting works

Conceptually, the splitter might try separators in this order:

```text
1. Paragraph
2. Line break
3. Sentence
4. Space
5. Character
```

The idea is:

> **Use the largest meaningful boundary possible, and only split further when the chunk is still too large.**

For example:

```text
Large document
      ↓
Try paragraph boundaries
      ↓
Still too large?
      ↓
Split paragraphs
      ↓
Still too large?
      ↓
Try sentences
      ↓
Still too large?
      ↓
Try words/spaces
```

That's why it's called **recursive**.

---

# 12. Example of recursive splitting

Suppose your target chunk size is:

```text
100 tokens
```

You have:

```text
Paragraph 1 = 40 tokens
Paragraph 2 = 35 tokens
Paragraph 3 = 80 tokens
```

The splitter can create:

```text
Chunk 1:
Paragraph 1 + Paragraph 2
= 75 tokens
```

Then:

```text
Chunk 2:
Paragraph 3
= 80 tokens
```

That's much better than:

```text
Chunk 1:
First 100 tokens

Chunk 2:
Next 100 tokens
```

because the recursive approach tries to respect natural boundaries.

---

# 13. Why recursive splitting is a good default

For general-purpose RAG, recursive splitting is often a strong starting point because it balances:

```text
Size
+
Context
+
Natural boundaries
```

It's particularly useful for:

* PDFs after text extraction
* Markdown
* HTML
* Documentation
* Articles
* Policies
* Technical documents

---

# 14. Semantic chunking

Now we move to a more advanced approach.

Recursive splitting primarily cares about **text structure**.

Semantic chunking cares about **meaning**.

Suppose you have:

```text
Paragraph 1:
Employees can work remotely up to 3 days per week.

Paragraph 2:
Remote work requires manager approval.

Paragraph 3:
The company headquarters is located in Dubai.

Paragraph 4:
The Dubai office has 500 employees.
```

A semantic chunker tries to identify that:

```text
Paragraph 1 + Paragraph 2
```

are semantically related:

```text
Remote work policy
```

while:

```text
Paragraph 3 + Paragraph 4
```

belong to:

```text
Office information
```

So instead of only looking at:

```text
paragraph boundaries
```

it considers:

```text
semantic similarity
```

---

# 15. How semantic chunking works conceptually

Imagine:

```text
Paragraph 1 → embedding A
Paragraph 2 → embedding B
Paragraph 3 → embedding C
Paragraph 4 → embedding D
```

The system compares neighboring paragraphs:

```text
A ↔ B → highly similar
B ↔ C → very different
C ↔ D → highly similar
```

Therefore:

```text
Chunk 1:
Paragraph 1
Paragraph 2

Chunk 2:
Paragraph 3
Paragraph 4
```

The boundaries are based on **meaning**, not just size.

---

# 16. Recursive vs semantic chunking

This distinction is important.

### Recursive

Uses structural boundaries:

```text
paragraph
   ↓
sentence
   ↓
word
```

It's generally:

* Faster
* Cheaper
* Easier
* Predictable

### Semantic

Uses semantic relationships:

```text
meaning
   ↓
similarity
   ↓
semantic boundaries
```

It's generally:

* More sophisticated
* Potentially more expensive
* More computationally intensive
* Potentially better for complex documents

---

# 17. When should you use semantic chunking?

It's useful when document structure doesn't cleanly correspond to meaning.

For example:

```text
Technical documentation
Research papers
Long articles
Complex manuals
Mixed-topic documents
```

But don't assume:

> "Semantic chunking is always better."

It isn't automatically better.

For a clean Markdown document:

```text
# Authentication

## OAuth

...

# Database

## PostgreSQL

...
```

normal structural/recursive chunking may already work extremely well.

---

# 18. Metadata preservation

Now one of the most important parts of chunking.

Remember from the previous topic:

```text
Document
 ├── Content
 └── Metadata
```

Suppose your original document has:

```text
Document:

Content:
Employees can work remotely...

Metadata:
source = HR_Policy.pdf
page = 15
department = HR
```

You split it:

```text
Document
      ↓
Chunk 1
Chunk 2
Chunk 3
```

You don't want to lose:

```text
source
page
department
```

Each chunk should retain the relevant metadata.

Conceptually:

```text
Chunk 1
├── Content
└── Metadata
    ├── source = HR_Policy.pdf
    ├── page = 15
    └── department = HR

Chunk 2
├── Content
└── Metadata
    ├── source = HR_Policy.pdf
    ├── page = 16
    └── department = HR
```

---

# 19. Why metadata preservation matters

Suppose the user asks:

> What is the remote-work policy?

Your retriever finds:

```text
Chunk 37
```

The chunk says:

```text
Employees can work remotely up to 3 days per week.
```

But you also want:

```text
Source: HR_Policy.pdf
Page: 15
```

Without metadata, your application might not know where that chunk came from.

Metadata is also useful for filtering:

```text
department = HR
```

or:

```text
documentType = policy
```

or:

```text
year = 2026
```

---

# 20. Metadata + filtering

This becomes especially powerful in larger RAG systems.

Imagine your vector store contains:

```text
HR policies
Engineering documentation
Finance documents
Security documents
```

The user asks:

> What is our remote work policy?

You can potentially do:

```text
Metadata filter:
department = HR
```

then:

```text
Semantic search:
"remote work policy"
```

So:

```text
                 Query
                   ↓
          Metadata filtering
                   ↓
             HR documents
                   ↓
          Vector similarity
                   ↓
           Relevant chunks
```

This is much better than searching your entire company's knowledge base blindly.

---

# 21. Chunk metadata can be richer

You might have:

```json
{
  "source": "HR_Policy.pdf",
  "page": 15,
  "documentType": "policy",
  "department": "HR",
  "version": "2026.1",
  "section": "Remote Work",
  "language": "en"
}
```

Then you can build sophisticated retrieval rules.

For example:

```text
documentType = "policy"
AND
department = "HR"
AND
version = "2026.1"
```

followed by semantic similarity search.

---

# 22. Parent-child context

Here's an advanced concept worth understanding now.

Sometimes you want:

```text
Small chunk for retrieval
```

but:

```text
Larger context for the LLM
```

For example:

```text
Parent document
       ↓
Section
       ↓
Small child chunks
```

You search using the small child chunk:

```text
Child chunk:
"3 days per week"
```

But when you find it, you give the LLM the larger section:

```text
Remote Work Policy

Eligibility:
All full-time employees...

Remote Days:
Employees can work remotely up to 3 days per week.

Approval:
Manager approval is required...
```

This gives you:

```text
Precise retrieval
+
Rich context
```

You don't need to implement this yet, but it's an important idea for advanced RAG.

---

# 23. How chunking affects retrieval

This is probably the **most important takeaway** from this topic.

Suppose the user asks:

> How many days can employees work remotely?

### Bad chunking

```text
Chunk:
"Employees can work remotely..."
```

Maybe the number `3` is in the next chunk.

Retrieval may return only:

```text
Employees can work remotely...
```

The LLM doesn't have the complete answer.

---

### Better chunking

```text
Employees can work remotely up to 3 days per week.
```

One semantic unit.

Retrieval gets the entire fact.

---

### Even better

```text
Remote Work Policy

Employees can work remotely up to 3 days per week.
Remote work requires manager approval.
```

Now the LLM has both:

```text
Answer
+
Relevant condition
```

That's why chunking is not merely a technical preprocessing step.

> **Chunking directly affects the quality of your RAG answers.**

---

# 24. How chunk size and overlap interact

Think about:

```text
Chunk Size
    +
Chunk Overlap
    +
Splitting Strategy
```

For example:

```text
Chunk size = 500 tokens
Overlap = 50 tokens
Strategy = Recursive
```

means:

```text
          500 tokens
┌─────────────────────────┐
│       Chunk 1           │
└─────────────────────────┘
                  ┌─────────────────────────┐
                  │       Chunk 2           │
                  └─────────────────────────┘
                    ↑
                  50 token
                  overlap
```

You shouldn't choose these numbers blindly.

You should test them against your actual documents and retrieval quality.

---

# 25. Your `TokenTextSplitter` example

You've already used:

```java
TokenTextSplitter.builder()
        .withChunkSize(...
        .withMaxNumChunks(400)
```

The important concept is:

```text
TokenTextSplitter
        ↓
Document
        ↓
Chunks
```

For your HR PDF, you could conceptually have:

```text
HR Policy
   ↓
TikaDocumentReader
   ↓
Documents
   ↓
TokenTextSplitter
   ↓
Chunk 1
Chunk 2
Chunk 3
...
   ↓
EmbeddingModel
   ↓
pgvector
```

So what you've already implemented isn't just random preprocessing — you're implementing a core RAG pipeline.

---

# 26. A practical strategy for your applications

For a first production-quality RAG system, I'd think about chunking in this order:

### Level 1 — Start simple

```text
Recursive splitting
+
reasonable chunk size
+
small overlap
```

### Level 2 — Preserve structure

Try to keep:

```text
Heading
+
paragraphs belonging to heading
```

together.

### Level 3 — Preserve metadata

Every chunk should know:

```text
source
page
section
document type
etc.
```

### Level 4 — Evaluate

Ask:

```text
Did the correct chunk get retrieved?
```

before worrying about sophisticated algorithms.

### Level 5 — Semantic chunking

Introduce it when normal splitting isn't giving good retrieval results.

---

# 27. A useful mental model

Think of a document like a book.

You don't want:

```text
Entire book → one chunk
```

And you don't want:

```text
Every word → one chunk
```

You want something like:

```text
Book
 │
 ├── Chapter
 │     ├── Section
 │     │     ├── Paragraph
 │     │     └── Paragraph
 │     │
 │     └── Section
 │           └── Paragraph
 │
 └── Chapter
```

The goal of chunking is to create pieces that are:

**Small enough to retrieve precisely, but large enough to make sense on their own.**

---

# 28. The complete RAG ingestion pipeline

You can now connect Topics 28/29 with your previous topics:

```text
                 DOCUMENT INGESTION
                         │
              PDF / Markdown / HTML
                         ↓
                 Document Reader
                         ↓
                     Document
                  content + metadata
                         ↓
                    CHUNKING
                         │
             ┌───────────┼───────────┐
             ↓           ↓           ↓
          Chunk Size  Overlap   Splitting
                                  Strategy
                                     ↓
                         Recursive / Semantic
                                     ↓
                         Chunks + Metadata
                                     ↓
                                Embeddings
                                     ↓
                           PostgreSQL + pgvector
```

Then at query time:

```text
User Question
      ↓
Query Embedding
      ↓
Metadata filtering (optional)
      ↓
Similarity Search
      ↓
Relevant Chunks
      ↓
Prompt + Context
      ↓
LLM
      ↓
Answer
```

## What you should remember

| Concept                   | Meaning                                                               |
| ------------------------- | --------------------------------------------------------------------- |
| **Chunk size**            | How much content goes into each chunk                                 |
| **Chunk overlap**         | Repeated content between adjacent chunks to preserve boundary context |
| **Recursive splitting**   | Splits using increasingly smaller natural boundaries                  |
| **Semantic chunking**     | Creates boundaries based on meaning/semantic similarity               |
| **Metadata preservation** | Keeps source/page/section/etc. attached to chunks                     |
| **Goal of chunking**      | Precise retrieval **without losing context**                          |

The single most important sentence for this topic is:

> **Good chunking creates self-contained, semantically meaningful pieces that are small enough for accurate retrieval but large enough to preserve the context needed to answer the question.**

And one subtle point to carry forward: **there is no universally "best" chunk size**. A 200-token chunk might be excellent for FAQ-style documents but terrible for a technical API specification. In real RAG systems, chunking is something you **evaluate experimentally**, not just configure once.

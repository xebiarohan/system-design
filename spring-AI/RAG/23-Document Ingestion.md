# Document Ingestion

The goal of document ingestion is:

```text
Raw data
   ↓
Read / Extract
   ↓
Spring AI Document
   ↓
Chunking
   ↓
Embedding
   ↓
Vector Store
```

You already learned chunking, embeddings, and vector stores. **Document ingestion is what happens before those steps.**

---

# 1. What is a Document?

In Spring AI, a `Document` is essentially a piece of text plus information describing where that text came from.

Conceptually:

```java
Document document = new Document(
    "Employees can work remotely up to 3 days per week."
);
```

A document can also contain **metadata**:

```java
Document document = new Document(
    "Employees can work remotely up to 3 days per week.",
    Map.of(
        "source", "HR_Policy.pdf",
        "page", 5,
        "department", "HR"
    )
);
```

So think of a Spring AI `Document` as:

```text
Document
 ├── Text/content
 └── Metadata
```

This is important because the original source could be:

* PDF
* Markdown
* HTML
* Word document
* database record
* web page
* etc.

Spring AI wants to eventually turn these different sources into a common representation.

---

# 2. Why do we need Document Readers?

Suppose you have:

```text
HR_Policy.pdf
```

The LLM doesn't directly consume a PDF file as part of your RAG pipeline.

You first need to extract its useful content:

```text
HR_Policy.pdf
       ↓
Document Reader
       ↓
Spring AI Document
```

A **Document Reader** is responsible for reading a particular source and converting it into Spring AI `Document` objects.

For example:

```text
PDF
 ↓
PDF reader
 ↓
Document


Markdown
 ↓
Markdown reader
 ↓
Document


HTML
 ↓
HTML reader
 ↓
Document
```

This gives you a standard pipeline regardless of the original format.

---

# 3. PDF

PDFs are extremely common in RAG applications.

For example:

```text
HR_Policy.pdf
Product_Manual.pdf
Technical_Architecture.pdf
User_Guide.pdf
```

You already used:

```java
TikaDocumentReader
```

from your HR policy example.

Conceptually:

```java
TikaDocumentReader reader =
        new TikaDocumentReader("HR_Policy.pdf");

List<Document> documents = reader.get();
```

The reader extracts the text from the PDF.

For example, the PDF might contain:

```text
Page 1:
Company Overview

Page 2:
Remote Work Policy

Page 3:
Leave Policy
```

The reader turns that into Spring AI documents.

Depending on the reader and source, metadata such as the source and page-related information may also be available.

---

# 4. Why PDF ingestion isn't simply "extract all text"?

Because PDFs are primarily designed for **visual layout**, not necessarily semantic structure.

Imagine a PDF displaying:

```text
Employee          Leave Days
-----------------------------
John              20
David             25
Sarah             22
```

The extracted text might not always preserve the exact visual structure.

Or consider:

```text
Page 10

        Important Security Policy

        All passwords must...
```

Extraction tools need to reconstruct the meaningful text from the PDF's underlying representation.

That's why PDF ingestion can sometimes be surprisingly tricky.

For simple PDFs, libraries such as Apache Tika work very well.

---

# 5. Markdown

Markdown is much easier to work with because it is already text.

Example:

```markdown
# Remote Work Policy

## Eligibility

Employees can work remotely.

## Working Days

Employees can work remotely up to 3 days per week.

## Approval

Manager approval is required.
```

A Markdown reader can turn this into a Spring AI `Document`.

The advantage is that Markdown already has useful semantic structure:

```text
# → Heading
## → Subheading
- → List
**text** → Bold
```

That structure can potentially help later with chunking and retrieval.

For example, you might eventually want a chunk to retain:

```text
Remote Work Policy
→ Eligibility
→ Working Days
```

rather than treating the entire file as meaningless plain text.

---

# 6. HTML

HTML is another common RAG source.

For example:

```html
<html>
  <body>
    <h1>Remote Work Policy</h1>

    <h2>Working Days</h2>

    <p>
      Employees can work remotely up to 3 days per week.
    </p>
  </body>
</html>
```

A document reader extracts the meaningful content:

```text
Remote Work Policy

Working Days

Employees can work remotely up to 3 days per week.
```

rather than giving the LLM a bunch of:

```html
<div>
<span>
<p>
<h1>
```

HTML ingestion becomes particularly useful for:

* company documentation
* product documentation
* knowledge bases
* websites
* internal portals

---

# 7. Document vs Document Reader

This distinction is worth remembering.

### Document

Represents the **data after ingestion**.

```text
Document
 ├── content
 └── metadata
```

### Document Reader

Responsible for **reading the original source**.

```text
PDF
 ↓
Reader
 ↓
Document
```

So:

> **Reader = how you get the data in.**
> **Document = the standardized representation of the data.**

---

# 8. Metadata

Metadata is one of the most important parts of document ingestion.

Suppose you have:

```text
HR_Policy.pdf
```

with:

```text
Page 15:
Remote work policy...
```

Your actual document content could be:

```text
Employees can work remotely up to 3 days per week.
```

Metadata could be:

```json
{
  "source": "HR_Policy.pdf",
  "page": 15,
  "department": "HR",
  "documentType": "policy"
}
```

So:

```text
Document
│
├── Content
│   └── Employees can work remotely...
│
└── Metadata
    ├── source = HR_Policy.pdf
    ├── page = 15
    ├── department = HR
    └── documentType = policy
```

---

# 9. Why is metadata important in RAG?

Because you can use it during **retrieval**.

Imagine your vector database contains:

```text
HR Policy
Engineering Policy
Finance Policy
Security Policy
```

User asks:

```text
What is our engineering deployment policy?
```

Instead of searching everything, you can potentially filter:

```text
department = "Engineering"
```

Then perform semantic search only within relevant documents.

Conceptually:

```text
User Question
      ↓
Metadata Filter
department = Engineering
      ↓
Vector Similarity Search
      ↓
Relevant chunks
```

This can improve both:

* performance
* retrieval accuracy

---

# 10. Metadata can also preserve document location

Imagine the LLM answers:

> Employees can work remotely up to 3 days per week.

You may want your application to show:

```text
Source: HR_Policy.pdf
Page: 15
```

That information can come from metadata.

This is extremely useful for **citations/source references** in RAG applications.

For example:

```text
Answer:
Employees can work remotely up to 3 days per week.

Source:
HR_Policy.pdf — Page 15
```

---

# 11. Metadata survives the RAG pipeline

This is an important mental model.

Initially:

```text
PDF
 ↓
Document
```

The document has:

```text
content + metadata
```

Then:

```text
Document
 ↓
Chunk
 ↓
Embedding
 ↓
Vector Store
```

The metadata can remain associated with the chunk/vector.

So later:

```text
Vector Search
      ↓
Document
      ↓
content + metadata
```

You don't just retrieve:

```text
"Employees can work remotely..."
```

You can retrieve:

```text
Content:
Employees can work remotely...

Metadata:
source = HR_Policy.pdf
page = 15
department = HR
```

That's very valuable.

---

# 12. Multiple documents

A real RAG system won't have just one PDF.

Imagine:

```text
documents/
├── hr-policy.pdf
├── security-policy.pdf
├── travel-policy.pdf
├── engineering-guide.md
└── deployment-guide.html
```

Your ingestion pipeline becomes:

```text
PDF ──────────┐
Markdown ─────┤
HTML ─────────┤
               ↓
         Document Readers
               ↓
       Spring AI Documents
               ↓
            Chunking
               ↓
          Embeddings
               ↓
          Vector Store
```

This is why the `Document` abstraction is useful: **different source formats eventually become a common representation.**

---

# 13. Your existing HR PDF example

Your previous example was essentially:

```text
HR_Policy.pdf
       ↓
TikaDocumentReader
       ↓
List<Document>
       ↓
TokenTextSplitter
       ↓
Chunks
       ↓
EmbeddingModel
       ↓
PostgreSQL + pgvector
```

So let's map it directly to your roadmap:

| Topic           | Your HR example       |
| --------------- | --------------------- |
| Document        | Spring AI `Document`  |
| Document Reader | `TikaDocumentReader`  |
| PDF             | `HR_Policy.pdf`       |
| Chunking        | `TokenTextSplitter`   |
| Embedding       | `EmbeddingModel`      |
| Vector DB       | PostgreSQL + pgvector |
| Retrieval       | Similarity search     |
| Generation      | Chat model            |

You're now learning the **ingestion half** of the pipeline.

---

# 14. Document ingestion vs chunking

Don't mix these two.

### Document ingestion

Answers:

> **How do I convert my source into usable documents?**

```text
PDF
Markdown
HTML
      ↓
Document
```

### Chunking

Answers:

> **How do I split those documents into useful pieces?**

```text
Document
      ↓
Chunk 1
Chunk 2
Chunk 3
...
```

So:

```text
             INGESTION
                 ↓
        ┌────────────────┐
        │ Spring Document│
        └───────┬────────┘
                ↓
             CHUNKING
                ↓
          Smaller chunks
                ↓
           EMBEDDINGS
```

---

# 15. One practical example

Suppose you have this Markdown file:

```markdown
# Remote Work Policy

## Eligibility

All full-time employees are eligible.

## Remote Days

Employees may work remotely up to 3 days per week.

## Approval

Remote work must be approved by the employee's manager.
```

After ingestion:

```text
Document

Content:
# Remote Work Policy

## Eligibility

All full-time employees are eligible.

## Remote Days

Employees may work remotely up to 3 days per week.

## Approval

Remote work must be approved by the employee's manager.

Metadata:
source = remote-work-policy.md
type = policy
department = HR
```

Then chunking might produce:

```text
Chunk 1:
Remote Work Policy
Eligibility
All full-time employees are eligible.

Chunk 2:
Remote Days
Employees may work remotely up to 3 days per week.

Chunk 3:
Approval
Remote work must be approved by the employee's manager.
```

Then each chunk gets embedded and stored.

---

# 16. The bigger picture

At this point, your RAG architecture should look like:

```text
                 DOCUMENT INGESTION
                         │
       ┌─────────────────┼─────────────────┐
       ↓                 ↓                 ↓
      PDF             Markdown           HTML
       │                 │                 │
       └─────────────────┼─────────────────┘
                         ↓
                  Document Reader
                         ↓
                  Spring AI Document
                         │
                    + Metadata
                         ↓
                      Chunking
                         ↓
                     Embeddings
                         ↓
                  Vector Store
                         ↓
                 ─────────────────
                   User Question
                         ↓
                      Retrieval
                         ↓
                  Relevant Chunks
                         ↓
                  Prompt + Context
                         ↓
                        LLM
                         ↓
                       Answer
```

### What you should remember from this topic

**1. Document**
A standardized Spring AI representation containing content + metadata.

**2. Document Reader**
Reads/extracts content from sources such as PDF, Markdown, HTML.

**3. PDF**
Usually requires a parser/extractor such as Tika because PDF is layout-oriented.

**4. Markdown**
Already structured text and generally easy to ingest.

**5. HTML**
Useful for web/documentation content; reader extracts meaningful text.

**6. Metadata**
Information *about* the content—source, page, document type, department, URL, etc.—which becomes very useful for filtering and citations.

The key flow is:

```text
Source
  ↓
Reader
  ↓
Document (content + metadata)
  ↓
Chunk
  ↓
Embedding
  ↓
Vector Store
```

**Next, the natural topic after this is Chunking / Text Splitting**, where you'll learn *how to decide where a document should be split and why bad chunking can seriously hurt RAG retrieval quality*.

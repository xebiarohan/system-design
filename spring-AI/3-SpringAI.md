```markdown
# What is Spring AI?

## 1) The problem Spring AI solves

Imagine you have a **Spring Boot** application and you want to call an **LLM**.

Without Spring AI, you might directly integrate with an LLM provider:

```text
Spring Boot
     ↓
OpenAI API
     ↓
GPT model
```

In that case, your Java code becomes tightly coupled to the provider’s API. For example (conceptually):

```java
OpenAIClient client = new OpenAIClient(apiKey);

OpenAIResponse response =
    client.chat(...);
```

Now suppose six months later you want to switch:

- **OpenAI → Anthropic**
- **OpenAI → Google Gemini**
- **Cloud LLM → Ollama running locally**

Your application code may require significant changes.

**Spring AI** tries to put an **abstraction layer** between your application and the AI provider:

```text
Your Application
      │
      ↓
  ┌───────────┐
  │ Spring AI │
  └─────┬─────┘
        │
 ┌──────┼───────────────┐
 ↓      ↓               ↓
OpenAI Anthropic       Google
 ↓      ↓               ↓
Model  Model           Model
```

That is the fundamental idea.

---

## 2) Spring AI is NOT an AI model

This distinction is extremely important.

**Spring AI does not provide the LLM itself.**  
It is not:

- `Spring AI = GPT`

Instead:

- **Spring AI = framework for building applications that use AI models**

Think about it like this:

```text
Spring Boot
    ↓
Application framework

Spring AI
    ↓
AI application framework

OpenAI / Anthropic / Google / Ollama
    ↓
AI model providers

GPT / Claude / Gemini / Llama
    ↓
Actual models
```

For example:

```text
Your Java application
      ↓
   Spring AI
      ↓
 OpenAI API
      ↓
  GPT model
```

Spring AI is essentially the **integration/application layer**.

---

## 3) Why does Spring AI exist?

Spring developers already understand a major principle:

> Don't couple your application unnecessarily to infrastructure.

For example, Spring provides abstractions around things like:

```text
Database
   ↓
Spring Data
   ↓
Database implementation
```

You don’t want every piece of business logic to know which database driver is used.

Similarly, Spring AI provides abstractions around AI providers:

```text
Your application
       ↓
   Spring AI
       ↓
AI provider
```

The application works primarily with **Spring AI concepts**, not provider-specific concepts.

---

## 4) The AI provider problem

The AI ecosystem changes very quickly. You could have:

- OpenAI
- Anthropic
- Google
- Azure OpenAI
- Amazon Bedrock
- Ollama
- Mistral
- Groq
- etc.

Each provider can have different:

- APIs
- Authentication mechanisms
- Request/response formats
- Model names
- Configuration
- Streaming mechanisms
- Tool-calling mechanisms
- Embedding APIs

Without an abstraction, your application becomes heavily coupled to one provider.

Spring AI attempts to normalize many of these concepts.

---

## 5) Spring AI architecture (simplified view)

A simplified view looks like this:

```text
Spring Boot Application
         │
         ↓
   ┌───────────┐
   │ Spring AI │
   └───────────┘
         │
 ┌───────┼────────────────────┐
 ↓       ↓                    ↓
Chat Model Embedding Model  Image Model
    │         │                 │
    ↓         ↓                 ↓
 Provider   Provider         Provider
    │         │                 │
    ↓         ↓                 ↓
   LLM      Embedding    Image Model
```

---

## 6) Layer 1 — Your Spring Boot application

This is your normal application.

Example flow:

```text
Controller
    ↓
Service
    ↓
Spring AI
    ↓
LLM
```

For example:

```java
@RestController
public class ChatController {

    private final ChatService chatService;

    @GetMapping("/ask")
    public String ask(String question) {
        return chatService.ask(question);
    }
}
```

Ideally, the controller shouldn’t know:

- how OpenAI’s HTTP API works
- how Anthropic authentication works
- how OpenAI response JSON is structured
- how streaming is implemented by a specific provider

That’s infrastructure.

---

## 7) Layer 2 — Spring AI abstractions

This is where Spring AI becomes interesting.

Spring AI gives concepts such as:

- `ChatModel`
- `EmbeddingModel`
- `ChatClient`
- `Prompt`
- `Message`
- `VectorStore`
- `Advisor`
- `Tool`

You’ll study these in more detail later. For Topic 3, understand their role:

- `ChatModel` → abstraction for chat-oriented AI models
- `EmbeddingModel` → abstraction for generating embeddings

So your application can use a consistent programming model.

---

## 8) Model abstraction (most important concept)

Suppose your application needs a chat model:

```java
ChatModel model;
```

Your application doesn’t necessarily care whether the underlying implementation is:

- OpenAI
- Anthropic
- Google
- Ollama
- etc.

Conceptually:

```text
ChatModel
   │
┌──┼────────────┐
↓  ↓            ↓
OpenAI impl   Google impl  Ollama impl
↓  ↓            ↓
GPT          Gemini       Llama
```

This is an example of **provider abstraction**.

---

## 9) Provider abstraction (and its limits)

If you build initially using OpenAI:

```text
Application
    ↓
Spring AI
    ↓
OpenAI
```

Later switch to another provider:

```text
Application
    ↓
Spring AI
    ↓
Another provider
```

Ideally, application-level code doesn’t need major changes.

**But beware:** abstraction doesn’t mean all models behave identically.

Models differ in:

- vision support
- tool calling support
- context window size
- pricing and latency
- quality, etc.

Spring AI can normalize the *programming model*, but it cannot make every model equivalent.

---

## 10) Spring Boot integration

Spring AI is designed to fit naturally into the Spring ecosystem.

Normally, Spring Boot handles:

- Configuration
- Dependency injection
- Beans
- Auto-configuration
- Properties and profiles
- HTTP clients
- Observability
- Testing

Spring AI builds on these mechanisms.

So instead of manually constructing every AI client, you configure the provider and let Spring Boot create the needed beans.

---

## 11) What is auto-configuration?

Suppose you add a Spring AI provider dependency and configure:

```properties
spring.ai.openai.api-key=...
```

Then Spring Boot can detect the dependency + configuration and automatically create the correct beans.

Conceptually:

```text
Dependency + Configuration
          ↓
Spring Boot Auto-configuration
          ↓
AI-related beans
          ↓
Your application
```

---

## 12) Dependency injection

Because Spring AI integrates with Spring’s DI model, you can inject AI components into services.

Example:

```java
@Service
public class CustomerService {

    private final ChatModel chatModel;

    public CustomerService(ChatModel chatModel) {
        this.chatModel = chatModel;
    }
}
```

Key idea:

- Your service depends on an **abstraction**
- Spring provides the implementation

Cleaner than scattering provider-specific clients everywhere.

---

## 13) Spring AI and the Spring ecosystem

Spring AI is powerful because it fits into a larger Spring architecture:

```text
React
  ↓
Spring MVC
  ↓
Spring Service
  ↓
Spring AI
 ┌────────┼─────────┐
 ↓        ↓         ↓
LLM     Vector DB   Tools
 ↓        ↓         ↓
Provider PostgreSQL  APIs
```

You can combine it with:

- Spring Boot
- Spring Security
- Spring Data
- Spring Web
- Spring Actuator
- Micrometer
- Spring AI

---

## 14) Spring AI is more than an "LLM client"

A beginner might think:

> Spring AI is just a wrapper around OpenAI.

That’s too narrow.

Spring AI provides abstractions/infrastructure for patterns like:

### Chat
```text
Chat
  ↓
Chat Model
  ↓
LLM
```

### Embeddings
```text
Text
  ↓
EmbeddingModel
  ↓
Vector
```

### RAG
```text
Question
  ↓
Retrieve documents
  ↓
Relevant context
  ↓
LLM
  ↓
Answer
```

### Tool calling
```text
User
  ↓
LLM
  ↓
Tool
  ↓
Java method/API
  ↓
LLM
  ↓
Answer
```

### Memory
```text
Conversation
  ↓
Memory
  ↓
Relevant context
  ↓
LLM
  ↓
Answer
```

### Structured output
```text
LLM
  ↓
Structured response
  ↓
Java object
```

So Spring AI is better thought of as:

> A Spring-oriented framework for building AI-powered applications.

---

## 15) A useful mental model

As a backend developer, remember Spring AI like this:

- **Spring Data** → database abstraction
- **Spring Security** → security abstraction
- **Spring AI** → AI application abstraction

But don’t take it too literally. AI systems have additional concerns:

- models
- prompts
- tokens
- embeddings
- vector stores
- RAG
- memory
- tools
- streaming
- multimodal input
- evaluation
- observability

Spring AI provides abstractions/integrations around these.

---

## 16) Where does `ChatClient` fit?

(You’ll cover this more later.)

Think:

```text
Your application
      ↓
  ChatClient
      ↓
  ChatModel
      ↓
Provider implementation
      ↓
     LLM
```

Example:

```text
Controller
  ↓
Service
  ↓
ChatClient
  ↓
ChatModel
  ↓
OpenAI
  ↓
GPT
```

`ChatClient` is a convenient fluent API for interacting with chat models.

---

## 17) `ChatModel` vs `ChatClient`

This distinction is extremely important.

At a high level:

### `ChatModel`
- lower-level model abstraction
- “Talk to the underlying chat model”

### `ChatClient`
- higher-level developer-friendly API
- “Build an interaction with the model”

Conceptually:

```text
ChatClient
   ↓
ChatModel
   ↓
AI Provider
   ↓
LLM
```

---

## 18) `EmbeddingModel`

Same abstraction idea applies to embeddings.

Example: converting:

> “Spring AI is useful”

into a vector:

```text
[0.12, -0.42, 0.81, ...]
```

Spring AI gives:

```text
EmbeddingModel
       ↓
Provider
       ↓
Embedding model
       ↓
Vector
```

This becomes crucial later when you learn:

- Embeddings → VectorStore → RAG

---

## 19) Spring AI and RAG

One of the biggest reasons to learn Spring AI is building:

> “Chat with my company’s documents”

Eventually, the architecture often looks like:

```text
User Question
     ↓
Spring AI
     ↓
Embedding
     ↓
PostgreSQL/pgvector
     ↓
Relevant chunks
     ↓
Prompt
     ↓
LLM
     ↓
Answer
```

Spring AI provides abstractions for many parts of this architecture.

That’s why the roadmap goes:

Spring AI fundamentals → Chat → Embeddings → Vector databases → RAG

---

## 20) Spring AI does not remove the need to understand AI

You should not think:

> Spring AI handles AI, so you don’t need LLM fundamentals.

Quite the opposite.

Even if you use `ChatClient`, you still need to understand:

- tokens
- context window
- system prompts
- temperature
- embeddings
- RAG
- tool calling

That’s why the roadmap starts with:

- Phase 0: LLM fundamentals
- Phase 1: Spring AI fundamentals
- Phase 2: Chat models

---

## 21) What happens when a request is made? (conceptual flow)

Suppose your app receives:

`GET /ask?question=What is Spring AI?`

Conceptual architecture:

```text
Browser
  ↓
Spring Controller
  ↓
Spring Service
  ↓
ChatClient
  ↓
ChatModel
  ↓
Spring AI provider implementation
  ↓
OpenAI/Anthropic/etc.
  ↓
LLM
```

And back on the response path:

```text
LLM
 ↓
Provider implementation
 ↓
ChatModel
 ↓
ChatClient
 ↓
Service
 ↓
Controller
 ↓
HTTP response
 ↓
Browser
```

That layered architecture should be in your head.

---

## 22) Why not just use the provider SDK?

You can absolutely use provider SDKs.

For a tiny app:

```text
Spring Boot
   ↓
OpenAI SDK
   ↓
OpenAI
```

might be perfectly fine.

Spring AI becomes more valuable as you need broader AI concerns:

- multiple providers
- prompt management
- structured output
- streaming
- embeddings
- vector stores
- RAG
- memory
- tool calling
- advisors
- observability

The value is not just “send an HTTP request to an LLM”.  
It’s having a Spring-native programming model for AI applications.

---

## 23) The abstraction has a trade-off

Abstraction comes with a trade-off:

- Direct SDK → maximum provider-specific control
- Spring AI → common abstraction + Spring integration

If a provider feature isn’t exposed by Spring AI, you may need provider-specific APIs.

A good engineer understands both:

- Spring AI abstraction
- underlying provider API

---

## 24) What you should understand after Topic 3

By the end of Topic 3, you should be able to answer:

### Q1. What is Spring AI?
A Spring framework for building AI-powered applications, providing abstractions and integrations for AI models and AI application patterns.

### Q2. Is Spring AI an LLM?
No.  
**Spring AI ≠ LLM**

### Q3. Who provides the actual model?
Providers such as:

- OpenAI
- Anthropic
- Google
- Ollama
- Azure
- Amazon Bedrock
- etc.

### Q4. Why use Spring AI?
Primarily:

- Abstraction
- Spring Boot integration
- Provider integrations
- AI application features

### Q5. What is auto-configuration?
Spring Boot can automatically create/configure relevant Spring AI infrastructure based on dependencies and application config.

### Q6. What is model abstraction?
Your application can program against concepts like:

- `ChatModel`
- `EmbeddingModel`

instead of coupling everything to a particular provider.

### Q7. Does Spring AI make all AI models equivalent?
No. Providers/models differ in:

- capabilities
- context limits
- pricing
- quality
- latency
- tool support
- vision support

---

## 25) The architecture you should memorize

If you remember only one diagram from Topic 3, remember this:

```text
┌─────────────────────┐
│   Spring Boot App   │
│                     │
│ Controller          │
│ Service             │
│ Business Logic      │
└──────────┬──────────┘
           │
           ↓
┌─────────────────────┐
│      Spring AI      │
│                     │
│ ChatClient          │
│ ChatModel           │
│ EmbeddingModel      │
│ Prompt              │
│ Advisors            │
│ VectorStore etc.    │
└──────────┬──────────┘
           │
      ┌────┼────────────┐
      ↓     ↓            ↓
   OpenAI  Anthropic     Google
      ↓     ↓            ↓
     GPT   Claude       Gemini
```

And eventually:

```text
                       Spring AI
                           │
       ┌───────────────────┼──────────────────┐
       ↓                   ↓                  ↓
    Chat/LLM            Embeddings          Tools
       │                   │                  │
       ↓                   │                  ↓
      RAG              Vector Store        APIs
       │                   │                  │
       └───────────────────┼──────────────────┘
                           ↓
                    AI Application
```

---

## 26) How to spend your ~2 hours

Don’t spend the entire two hours reading documentation.

- **First 30 minutes — Architecture**
  - Spring Boot → Spring AI → Provider → Model
  - Understand why the abstraction exists

- **Next 30 minutes — Core concepts**
  - Learn roles of:
    - `ChatModel`
    - `EmbeddingModel`
    - `ChatClient`
    - `Prompt`
    - `Message`
  - Don’t memorize methods yet

- **Next 30 minutes — Hands-on**
  - Create a tiny Spring Boot app
  - Make one LLM request
  - Goal:
    - HTTP request → Controller → ChatClient → LLM → Response

- **Final 30 minutes — Experiment**
  - Change provider/model configuration (if possible)
  - Ask:
    - What belongs to my app?
    - What belongs to the AI provider?

---

## The most important takeaway

As a backend engineer, summarize Topic 3 like this:

> **Spring AI** is an abstraction and integration layer that lets Spring Boot applications work with AI models and AI application capabilities using a **Spring-native programming model**, while reducing unnecessary coupling to individual AI providers.

Mental model:

```text
YOUR BUSINESS LOGIC
       ↓
  Spring AI
       ↓
┌─────────────┼─────────────┐
↓             ↓             ↓
OpenAI       Anthropic      Google
↓             ↓             ↓
GPT          Claude        Gemini
```

And that becomes the foundation for everything that follows in the roadmap.

---

## Where to go next in your roadmap

Topic 3 → Topic 4 → ChatClient/ChatModel → Structured Output → Embeddings → Vector Stores → RAG → Advisors → Memory → Tool Calling → MCP → Agents
```

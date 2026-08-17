3. What is Spring AI?
1. The problem Spring AI solves

Imagine you have a Spring Boot application and you want to call an LLM.

Without Spring AI, you might directly integrate with an LLM provider:

Spring Boot
     ↓
OpenAI API
     ↓
GPT model


Your Java code becomes tied to the provider's API.

For example, conceptually:

OpenAIClient client = new OpenAIClient(apiKey);

OpenAIResponse response =
    client.chat(...);


Now suppose six months later you want to switch:

OpenAI → Anthropic


or:

OpenAI → Google Gemini


or:

Cloud LLM → Ollama running locally


Your application code may need significant changes.

Spring AI tries to put an abstraction layer between your application and the AI provider:

                    Your Application
                          │
                          ↓
                    ┌───────────┐
                    │ Spring AI │
                    └─────┬─────┘
                          │
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
       OpenAI          Anthropic       Google
          │               │               │
          ↓               ↓               ↓
        Model            Model           Model


That's the fundamental idea.

2. Spring AI is NOT an AI model

This distinction is extremely important.

Spring AI does not provide the LLM itself.

It is not:

Spring AI = GPT


Instead:

Spring AI = framework for building applications that use AI models


Think about the relationship like this:

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


For example:

Your Java application
        ↓
Spring AI
        ↓
OpenAI API
        ↓
GPT model


Spring AI is essentially the integration/application layer.

3. Why does Spring AI exist?

Spring developers already understand a major principle:

Don't couple your application unnecessarily to infrastructure.

For example, Spring provides abstractions around things like:

Database
   ↓
Spring Data
   ↓
Database implementation


You don't necessarily want every piece of business logic to know the details of a particular database driver.

Similarly, Spring AI provides abstractions around AI providers:

Your application
       ↓
   Spring AI
       ↓
AI provider


The application works primarily with Spring AI concepts rather than provider-specific concepts.

4. The AI provider problem

The AI ecosystem is changing extremely quickly.

You could have:

OpenAI
Anthropic
Google
Azure OpenAI
Amazon Bedrock
Ollama
Mistral
Groq
etc.


And each provider can have different:

APIs
Authentication mechanisms
Request formats
Response formats
Model names
Configuration
Streaming mechanisms
Tool-calling mechanisms
Embedding APIs

Without an abstraction, your application can become heavily coupled to one provider.

Spring AI attempts to normalize many of these concepts.

5. Spring AI architecture

A simplified view looks like this:

                 Spring Boot Application
                          │
                          ↓
                    ┌───────────┐
                    │ Spring AI │
                    └───────────┘
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
   Chat Model       Embedding Model      Image Model
       │                  │                  │
       ↓                  ↓                  ↓
    Provider           Provider           Provider
       │                  │                  │
       ↓                  ↓                  ↓
      LLM             Embedding          Image Model


There are several important layers here.

6. Layer 1 — Your Spring Boot application

This is your normal application.

For example:

Controller
    ↓
Service
    ↓
Spring AI
    ↓
LLM


Imagine you have:

@RestController
public class ChatController {

    private final ChatService chatService;

    @GetMapping("/ask")
    public String ask(String question) {
        return chatService.ask(question);
    }
}


Your controller shouldn't ideally know:

how OpenAI's HTTP API works
how Anthropic authentication works
how an OpenAI response JSON is structured
how streaming is implemented by a particular provider

That's infrastructure.

7. Layer 2 — Spring AI abstractions

This is where Spring AI becomes interesting.

Spring AI gives you concepts such as:

ChatModel
EmbeddingModel
ChatClient
Prompt
Message
VectorStore
Advisor
Tool


You'll study these in much more detail in Topic 4 and later phases.

For Topic 3, understand the role of these concepts.

For example:

ChatModel
    ↓
Abstraction for interacting with chat-oriented AI models


and:

EmbeddingModel
    ↓
Abstraction for generating embeddings


So your application can work with a consistent programming model.

8. Model abstraction

This is probably the most important concept in Topic 3.

Suppose your application needs a chat model.

Conceptually:

ChatModel model;


Your application doesn't necessarily care whether the underlying implementation is:

OpenAI
Anthropic
Google
Ollama
etc.


You can think of it as:

                  ChatModel
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
    OpenAI impl   Google impl   Ollama impl
        │            │            │
        ↓            ↓            ↓
      GPT          Gemini        Llama


This is an example of provider abstraction.

9. Provider abstraction

Let's say you initially build:

Application
    ↓
Spring AI
    ↓
OpenAI


Later, you decide to use another provider:

Application
    ↓
Spring AI
    ↓
Another provider


Ideally, the application-level code doesn't need to fundamentally change.

That's one of Spring AI's major benefits.

But be careful:

Abstraction does not mean every provider behaves identically.

Different models still have different capabilities.

For example:

Model A → supports vision
Model B → doesn't

Model A → supports tool calling
Model B → doesn't

Model A → 128K context
Model B → 32K context


Spring AI can provide a common programming model, but it cannot magically make every model equivalent.

10. Spring Boot integration

The second major idea is:

Spring AI is designed to fit naturally into the Spring Boot ecosystem.

This is important if you're already a Spring developer.

Normally, you expect Spring Boot to handle things like:

Configuration
Dependency Injection
Beans
Auto-configuration
Properties
Profiles
HTTP clients
Observability
Testing


Spring AI builds on these mechanisms.

So instead of manually constructing every AI client, you can configure the provider and let Spring Boot create the necessary infrastructure.

Conceptually:

application.properties
        ↓
Spring Boot
        ↓
Auto-configuration
        ↓
Spring AI beans
        ↓
Your application

11. What is auto-configuration?

This is another important Topic 3 concept.

Suppose you add a Spring AI provider dependency.

You configure something like:

spring.ai.openai.api-key=...


Spring Boot can detect the relevant dependency and configuration and automatically create the appropriate beans.

Conceptually:

Dependency
    +
Configuration
    ↓
Spring Boot Auto-configuration
    ↓
AI-related beans
    ↓
Your application


This is the same philosophy you're already familiar with from Spring Boot.

For example, Spring Boot can automatically configure:

DataSource
ObjectMapper
RestClient
WebClient
Security infrastructure


depending on what dependencies and configuration you provide.

Spring AI extends this experience into AI integrations.

12. Dependency Injection

Because Spring AI fits into Spring's dependency injection model, you can inject AI components into your services.

Conceptually:

@Service
public class CustomerService {

    private final ChatModel chatModel;

    public CustomerService(ChatModel chatModel) {
        this.chatModel = chatModel;
    }
}


The important thing isn't memorizing this code yet.

The important architectural idea is:

Your Service
     ↓
depends on abstraction
     ↓
ChatModel
     ↓
Spring provides implementation


This is much cleaner than scattering provider-specific clients throughout your application.

13. Spring AI and the Spring ecosystem

This is where Spring AI becomes especially powerful.

It's not an isolated library.

It can sit inside a larger Spring architecture:

                    React
                      ↓
                 Spring MVC
                      ↓
                Spring Service
                      ↓
                  Spring AI
             ┌────────┼─────────┐
             ↓        ↓         ↓
           LLM      Vector DB   Tools
             ↓        ↓         ↓
          Provider  PostgreSQL  APIs


And you can combine this with things you already know:

Spring Boot
Spring Security
Spring Data
Spring Web
Spring Actuator
Micrometer
Spring AI


This is one reason Spring AI is attractive for Java backend developers.

14. Spring AI is more than "an LLM client"

This distinction is worth understanding early.

A beginner might think:

"Spring AI is just a wrapper around OpenAI."

That's too narrow.

Spring AI provides abstractions and infrastructure for several AI application patterns.

For example:

Chat
Application
    ↓
Chat Model
    ↓
LLM

Embeddings
Text
 ↓
EmbeddingModel
 ↓
Vector

RAG
Question
    ↓
Retrieve documents
    ↓
Relevant context
    ↓
LLM

Tool calling
User
 ↓
LLM
 ↓
Tool
 ↓
Java method/API
 ↓
LLM

Memory
Conversation
 ↓
Memory
 ↓
Relevant context
 ↓
LLM

Structured output
LLM
 ↓
Structured response
 ↓
Java object


So Spring AI is better thought of as:

A Spring-oriented framework for building AI-powered applications.

15. A useful mental model

As a backend developer, I'd remember Spring AI using this analogy:

Spring Data
     ↓
Database abstraction

Spring Security
     ↓
Security abstraction

Spring AI
     ↓
AI application abstraction


But don't take this analogy too literally.

Spring AI isn't simply "Spring Data for AI."

AI systems have additional concerns:

Models
Prompts
Tokens
Embeddings
Vector stores
RAG
Memory
Tools
Streaming
Multimodal input
Evaluation
Observability


Spring AI provides abstractions and integrations around these concerns.

16. Where does ChatClient fit?

You'll study ChatClient in Topic 4, but you should already understand its position.

Think:

Your application
       ↓
   ChatClient
       ↓
   ChatModel
       ↓
Provider implementation
       ↓
      LLM


For example:

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


The ChatClient gives you a convenient fluent API for interacting with chat models.

We'll go much deeper into this in Topic 4.

17. ChatModel vs ChatClient

This distinction is extremely important for your roadmap.

At a high level:

ChatModel

Lower-level model abstraction.

Think:

"Talk to the underlying chat model."

ChatClient

Higher-level developer-friendly API.

Think:

"Build an interaction with the model."


Conceptually:

                    ChatClient
                        ↓
                    ChatModel
                        ↓
                   AI Provider
                        ↓
                       LLM


Don't worry about memorizing every method yet.

You'll spend several hours on this in the next topic.

18. EmbeddingModel

The same abstraction idea applies to embeddings.

Suppose you want to convert:

"Spring AI is useful"


into:

[0.12, -0.42, 0.81, ...]


You could directly use a provider's embedding API.

Spring AI instead gives you an abstraction:

EmbeddingModel
       ↓
Provider
       ↓
Embedding model
       ↓
Vector


This becomes extremely important later when you learn:

Embeddings
     ↓
Vector Store
     ↓
RAG

19. Spring AI and RAG

One of the biggest reasons to learn Spring AI is building applications such as:

"Chat with my company's documents."

The architecture might eventually look like:

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


Spring AI provides abstractions for many pieces of this architecture.

That's why the roadmap moves from:

Spring AI fundamentals
        ↓
Chat
        ↓
Embeddings
        ↓
Vector databases
        ↓
RAG


The ordering is deliberate.

20. Spring AI doesn't remove the need to understand AI

This is a very important warning.

You should not think:

"Spring AI handles AI, so I don't need to understand LLMs."

Quite the opposite.

Suppose Spring AI gives you:

ChatClient


You still need to understand:

What is a token?
What is a context window?
What is a system prompt?
What is temperature?
What is an embedding?
What is RAG?
What is tool calling?


Otherwise you'll know the API but not understand the system.

That's why your roadmap correctly starts with:

Phase 0
LLM fundamentals
       ↓
Phase 1
Spring AI fundamentals
       ↓
Phase 2
Chat Models

21. What happens when a request is made?

Let's follow a conceptual request.

Suppose your application receives:

GET /ask?question=What is Spring AI?


The architecture might be:

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


Then the response travels back:

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


That layered architecture is what you should have in your head.

22. Why not just use the provider SDK?

This is a very reasonable question.

You absolutely can.

For a tiny application:

Spring Boot
    ↓
OpenAI SDK
    ↓
OpenAI


might be perfectly reasonable.

Spring AI becomes more valuable when your application starts dealing with broader AI application concerns:

Multiple providers
       +
Prompt management
       +
Structured output
       +
Streaming
       +
Embeddings
       +
Vector stores
       +
RAG
       +
Memory
       +
Tool calling
       +
Advisors
       +
Observability


The value isn't merely:

"I can send an HTTP request to an LLM."

The value is having a Spring-native programming model for AI applications.

23. The abstraction has a trade-off

Don't blindly assume abstraction is always better.

There is a trade-off:

Direct provider SDK
        ↓
Maximum provider-specific control


versus:

Spring AI
        ↓
Common abstraction + Spring integration


If you need a provider-specific feature that Spring AI doesn't expose directly, you may need to use provider-specific capabilities.

So a good engineer understands both:

Spring AI abstraction
        +
Underlying provider API


rather than believing the abstraction completely hides the provider.

24. What you should understand after Topic 3

By the end of these two hours, you should be able to answer these questions without looking at documentation.

Q1. What is Spring AI?

A Spring framework for building AI-powered applications, providing abstractions and integrations for AI models and related application patterns.

Q2. Is Spring AI an LLM?

No.

Spring AI ≠ LLM


Spring AI integrates applications with LLMs and other AI capabilities.

Q3. Who provides the actual model?

Providers such as:

OpenAI
Anthropic
Google
Ollama
Azure
Amazon Bedrock
etc.

Q4. Why use Spring AI?

Primarily:

Abstraction
+
Spring Boot integration
+
Provider integrations
+
AI application features

Q5. What is auto-configuration?

Spring Boot can automatically create/configure relevant Spring AI infrastructure based on dependencies and application configuration.

Q6. What is model abstraction?

Your application can program against concepts such as:

ChatModel
EmbeddingModel


rather than directly coupling every part of the application to a particular provider.

Q7. Does Spring AI make all AI models equivalent?

No.

Providers/models still have different:

Capabilities
Context limits
Pricing
Quality
Latency
Tool support
Vision support

25. The architecture you should memorize

If you remember only one diagram from Topic 3, remember this:

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
                ┌────────────┼────────────┐
                ↓            ↓            ↓
             OpenAI      Anthropic      Google
                ↓            ↓            ↓
               GPT          Claude       Gemini


And eventually:

                       Spring AI
                           │
       ┌───────────────────┼──────────────────┐
       ↓                   ↓                  ↓
    Chat/LLM            Embeddings          Tools
       │                   │                  │
       ↓                   ↓                  ↓
      RAG              Vector Store        APIs
       │                   │                  │
       └───────────────────┼──────────────────┘
                           ↓
                    AI Application

26. How I recommend you spend your 2 hours

Don't spend the entire two hours reading documentation.

First 30 minutes — Architecture

Understand:

Spring Boot
     ↓
Spring AI
     ↓
Provider
     ↓
Model


And understand why the abstraction exists.

Next 30 minutes — Core concepts

Learn the roles of:

ChatModel
EmbeddingModel
ChatClient
Prompt
Message


Don't memorize methods yet.

Next 30 minutes — Hands-on

Create a tiny Spring Boot application and make one LLM request.

Your goal is simply:

HTTP request
     ↓
Controller
     ↓
ChatClient
     ↓
LLM
     ↓
Response

Final 30 minutes — Experiment

Change the provider/model configuration if possible.

Then ask yourself:

What code belongs to my application, and what code belongs to the AI provider?

That's the architectural insight you're trying to develop.

The most important takeaway

As a backend engineer, I would summarize Topic 3 like this:

Spring AI is an abstraction and integration layer that lets Spring Boot applications work with AI models and AI application capabilities using a Spring-native programming model, while reducing unnecessary coupling to individual AI providers.

The mental model is:

              YOUR BUSINESS LOGIC
                      ↓
                 Spring AI
                      ↓
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
     OpenAI        Anthropic      Google
        ↓             ↓             ↓
       GPT          Claude        Gemini


And that is the foundation for everything that follows in your roadmap:

Topic 3
What is Spring AI?
       ↓
Topic 4
Core abstractions
       ↓
ChatClient / ChatModel
       ↓
Structured Output
       ↓
Embeddings
       ↓
Vector Stores
       ↓
RAG
       ↓
Advisors
       ↓
Memory
       ↓
Tool Calling
       ↓
MCP
       ↓
Agents


You can copy this entire response into a file named:

spring-ai-topic-3-what-is-spring-ai.md


and it will be valid Markdown.

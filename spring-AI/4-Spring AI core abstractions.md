# Topic 4 — Spring AI Core Abstractions

## 1. Learning Goal

### 1.1 What You Will Learn

By the end of this topic, you should understand:

- `ChatModel`
- `EmbeddingModel`
- `ChatClient`
- `Prompt`
- `Message`
- `ChatResponse`

> **Goal:** Understand why these abstractions exist, how they relate to each other, and when to use each one.

## 2. Why Does Spring AI Need Abstractions?

Spring AI provides an abstraction layer between your application and AI providers.

```text
Spring Boot Application
        |
        v
     Spring AI
        |
        v
  AI Abstraction
        |
        v
Provider Implementation
        |
        v
    AI Provider


Without an abstraction layer, your application could become tightly coupled to a specific provider:

OpenAiClient client;


With Spring AI, your application can work with abstractions such as:

ChatModel chatModel;

3. Core Spring AI Abstractions
Abstraction	Purpose
ChatModel	Abstraction for interacting with a chat/generative model
EmbeddingModel	Converts input into vector embeddings
ChatClient	Higher-level fluent API for interacting with chat models
Prompt	Represents a structured request sent to a chat model
Message	Represents an individual conversational message
ChatResponse	Represents the model's response
4. The Big Picture
                         Your Application
                                |
                                v
                           ChatClient
                                |
                                v
                             Prompt
                                |
                                v
                            Messages
                                |
                                v
                           ChatModel
                                |
                                v
                       Provider Implementation
                                |
                                v
                              LLM


For embeddings:

Text
 |
 v
EmbeddingModel
 |
 v
Embedding Vector
 |
 v
Vector Database

5. ChatModel
5.1 What Is ChatModel?

ChatModel is the core abstraction for communicating with a chat or generative AI model.

Think of it as:

ChatModel represents something capable of taking a prompt and generating a response.

Conceptually:

Prompt
   |
   v
ChatModel
   |
   v
ChatResponse

5.2 Why Does ChatModel Exist?

Suppose your service directly depends on a provider-specific implementation:

private final SomeProviderChatModel chatModel;


Your application is now coupled to that provider.

Instead, Spring AI encourages dependency on the abstraction:

private final ChatModel chatModel;


Conceptually:

                       ChatModel
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
         Provider A    Provider B    Provider C


The application depends on the interface/abstraction while Spring AI handles the provider-specific implementation.

6. EmbeddingModel

EmbeddingModel solves a different problem.

ChatModel is primarily about generating responses.

EmbeddingModel is about converting information into vectors.

Text
 |
 v
EmbeddingModel
 |
 v
Vector


For example:

"Spring Boot is a Java framework."
                |
                v
        EmbeddingModel
                |
                v
[0.12, -0.42, 0.73, 0.19, ...]


These vectors are later used for:

Semantic search
Vector databases
RAG
Similarity comparison
7. ChatModel vs EmbeddingModel
	ChatModel	EmbeddingModel
Main purpose	Generate responses	Generate vector representations
Input	Prompt/messages	Text
Output	Chat response	Embedding vector
Typical use	Chat, generation	Search, similarity, RAG

The easiest way to remember this is:

ChatModel
    =
Generate content

EmbeddingModel
    =
Represent content as vectors

8. Message

A chat model does not simply receive one giant string.

Modern chat APIs work with structured messages.

For example:

SYSTEM:
You are an expert Java instructor.

USER:
Explain dependency injection.

ASSISTANT:
Dependency injection is a design technique...


Spring AI represents these conversational units using Message.

A message conceptually contains:

Message
 |
 +---- Role
 |
 +---- Content

8.1 System Message

Provides instructions or context:

System:
You are an expert Java instructor.
Explain concepts with simple examples.

8.2 User Message

Represents the user's request:

User:
Explain dependency injection.

8.3 Assistant Message

Represents the assistant/model response:

Assistant:
Dependency injection is a design technique...

9. Why Are Messages Separate Objects?

You might wonder:

Why not simply send a String?

Because a chat interaction has structure.

Instead of:

"Explain dependency injection."


you can have:

SYSTEM:
You are an expert Java teacher.

USER:
Explain dependency injection.


The model can distinguish between:

System instructions
        +
User request
        +
Conversation history


This structure becomes important for:

System prompts
Conversation history
Memory
Tool calling
Multimodal input
Prompt construction
10. Prompt

A Prompt represents the structured request sent to a chat model.

Conceptually:

Prompt
 |
 +---- Messages
 |
 +---- Model / request options


For example:

Prompt
 |
 +---- System Message
 |
 +---- User Message
 |
 +---- Options


Therefore:

Prompt != String


A better mental model is:

Prompt
 |
 +---- Messages
 |
 +---- Options

11. Prompt vs Message

This is a common beginner confusion.

Message

One individual conversational unit:

User:
Explain dependency injection.

Prompt

The complete structured request:

Prompt
 |
 +---- System Message
 |
 +---- User Message
 |
 +---- Options


Remember:

A Message is a part of the conversation. A Prompt represents the request being sent to the model.

12. ChatClient

ChatClient provides a higher-level fluent API for interacting with chat models.

For example:

String response = chatClient
        .prompt()
        .user("Explain dependency injection.")
        .call()
        .content();


This is easier than manually constructing every lower-level object.

13. ChatClient vs ChatModel

This is one of the most important distinctions in Spring AI.

ChatClient
    |
    | Higher-level API
    v
ChatModel
    |
    | Model abstraction
    v
Provider Implementation
    |
    v
LLM

ChatClient

Think:

"I want to build and execute an AI request conveniently."

ChatModel

Think:

"I need the abstraction representing the actual chat model."

They exist at different abstraction levels.

14. ChatClient Fluent API

A typical interaction:

String answer = chatClient
        .prompt()
        .system("You are an expert Java teacher.")
        .user("Explain dependency injection.")
        .call()
        .content();


Read it as:

Create a prompt
      |
      v
Set system instructions
      |
      v
Set user request
      |
      v
Call the model
      |
      v
Get the generated content

15. Understanding prompt()
chatClient.prompt()


Starts building an AI request.

Think:

ChatClient
    |
    v
"I want to create a prompt."

16. Understanding system()
chatClient
        .prompt()
        .system("""
            You are an expert Java instructor.
            Explain concepts with simple examples.
        """)


This adds system-level instructions.

17. Understanding user()
.user("Explain interfaces.")


This adds the user's request.

18. Understanding call()
.call()


This executes the request.

Conceptually:

Request constructed
        |
        v
      call()
        |
        v
       LLM

19. Understanding content()
String answer = chatClient
        .prompt()
        .user("What is dependency injection?")
        .call()
        .content();


content() extracts the generated textual content.

20. content() vs chatResponse()
content()

Use this mental model:

"I only care about the generated text."

String answer = chatClient
        .prompt()
        .user("Explain Spring.")
        .call()
        .content();

chatResponse()

Use this mental model:

"I want the richer model response."

Conceptually:

ChatResponse
 |
 +---- Result / generated message
 |
 +---- Metadata
 |
 +---- Usage information
 |
 +---- Other response information


The exact metadata depends on the Spring AI version and model/provider.

21. Complete ChatClient Example
@Service
public class AiService {

    private final ChatClient chatClient;

    public AiService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    public String explain(String topic) {

        return chatClient
                .prompt()
                .system("""
                    You are an expert Java instructor.
                    Explain concepts clearly with examples.
                    """)
                .user("Explain: " + topic)
                .call()
                .content();
    }
}


The important part is understanding this flow:

chatClient
    |
    v
prompt()
    |
    v
system()
    |
    v
user()
    |
    v
call()
    |
    v
content()

22. Complete Architecture
                         Spring Boot Application
                                  |
                                  v
                             ChatClient
                                  |
                                  | builds
                                  v
                                Prompt
                                  |
                                  | contains
                                  v
                              Messages
                           /      |      \
                          /       |       \
                         v        v        v
                     System     User   Assistant
                                  |
                                  v
                              ChatModel
                                  |
                                  v
                         Provider Adapter
                                  |
                                  v
                                LLM
                                  |
                                  v
                           ChatResponse
                                  |
                                  v
                              content()

23. Lower-Level vs Higher-Level API
Higher-Level API
String answer = chatClient
        .prompt()
        .user("Explain interfaces.")
        .call()
        .content();


Flow:

ChatClient
    |
    v
Prompt
    |
    v
ChatModel
    |
    v
LLM

Lower-Level API

Conceptually:

Prompt prompt = new Prompt(...);

ChatResponse response = chatModel.call(prompt);


Flow:

Prompt
   |
   v
ChatModel
   |
   v
LLM
   |
   v
ChatResponse

24. Why Have Both?

They solve different problems.

	ChatClient	ChatModel
Abstraction level	Higher	Lower
Main purpose	Convenient application API	Model abstraction
Request construction	Fluent	More explicit
Typical usage	Application code	Lower-level/custom integrations
Mental model	Build an AI request	Call the model

Think:

ChatClient
    |
    | Convenience
    v
ChatModel
    |
    | Abstraction
    v
Provider

25. Spring Boot Auto-Configuration

Spring AI integrates with Spring Boot's auto-configuration.

Conceptually:

application.yml
      |
      | Model/API configuration
      v
Spring Boot
      |
      v
Spring AI Auto-Configuration
      |
      +---- ChatModel
      |
      +---- ChatClient
      |
      +---- EmbeddingModel


Your application can inject these objects using dependency injection.

For example:

@Service
public class AiService {

    private final ChatClient chatClient;

    public AiService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }
}

26. Provider Independence

Suppose today your architecture is:

Application
    |
    v
ChatModel
    |
    v
Provider A


Later:

Application
    |
    v
ChatModel
    |
    v
Provider B


The application-level architecture can remain largely the same.

However, provider independence does not mean all providers are identical.

Providers can differ in:

Supported models
Context windows
Tool calling
Structured output
Multimodal capabilities
Model parameters
Tokenization
Pricing
Response metadata

Therefore:

Spring AI reduces provider coupling. It does not make all AI providers identical.

27. Why This Architecture Is Useful

You don't want your business logic filled with provider-specific code:

ProviderClient
ProviderRequest
ProviderResponse
ProviderException
ProviderJson
ProviderConfiguration
...


Instead:

Business Logic
      |
      v
Spring AI Abstraction
      |
      v
Provider Implementation
      |
      v
AI Provider


This creates better separation of concerns.

28. How This Connects to RAG

Later, when you study RAG, these abstractions become building blocks.

                         RAG Application
                                |
                 +--------------+--------------+
                 |                             |
                 v                             v
          EmbeddingModel                   ChatClient
                 |                             |
                 v                             v
           VectorStore                      Prompt
                 |                             |
                 v                             v
        Relevant Documents                 Messages
                 |                             |
                 +-------------+---------------+
                               |
                               v
                           ChatModel
                               |
                               v
                              LLM


A simplified RAG flow:

Question
   |
   +--------------------+
   |                    |
   v                    v
EmbeddingModel       ChatClient
   |                    |
   v                    v
Vector Search         Prompt
   |                    |
   v                    |
Relevant Chunks         |
   |                    |
   +---------+----------+
             |
             v
          ChatModel
             |
             v
             LLM

29. How This Connects to Memory

Memory introduces previous conversation messages into the model context.

Conceptually:

System Message
       +
Previous User Message
       +
Previous Assistant Message
       +
Current User Message
       |
       v
     Prompt
       |
       v
   ChatModel
       |
       v
      LLM


This is why understanding Message is important before learning chat memory.

30. How This Connects to Tool Calling

Later you'll learn tool/function calling.

The architecture becomes:

User
  |
  v
ChatClient
  |
  v
Prompt
  |
  v
ChatModel
  |
  v
LLM
  |
  +------------------+
  |                  |
  v                  v
Answer           Tool Call
                     |
                     v
                 Java Method
                     |
                     v
                   Result
                     |
                     v
                    LLM
                     |
                     v
                  Answer


Again, the core abstractions remain underneath.

31. Important Distinctions to Memorize
ChatClient vs ChatModel
ChatClient
    =
High-level fluent API

ChatModel
    =
Core model abstraction

Prompt vs Message
Message
    =
One conversational unit

Prompt
    =
Complete structured model request

ChatModel vs EmbeddingModel
ChatModel
    =
Generate content

EmbeddingModel
    =
Generate vector representation

content() vs chatResponse()
content()
    =
Give me the generated text

chatResponse()
    =
Give me the structured response

32. The Most Important Diagram

If you remember only one diagram from this topic, remember this:

                     YOUR APPLICATION
                            |
                            v
                       ChatClient
                            |
                            v
                         Prompt
                            |
                            v
                        Messages
                     /      |      \
                    /       |       \
                   v        v        v
               System     User    Assistant
                            |
                            v
                        ChatModel
                            |
                            v
                   Provider Adapter
                            |
                            v
                           LLM
                            |
                            v
                      ChatResponse


For embeddings:

Text
    |
    v
EmbeddingModel
    |
    v
Embedding Provider
    |
    v
Vector
    |
    v
Vector Store

33. What You Should Know After Topic 4

You should be able to answer these questions without looking at documentation.

Question 1 — What is ChatModel?

A core Spring AI abstraction representing interaction with a chat/generative AI model.

Question 2 — What is EmbeddingModel?

An abstraction for converting input such as text into vector embeddings, commonly used for semantic search and RAG.

Question 3 — What is Message?

A structured conversational unit such as a system, user, or assistant message.

Question 4 — What is Prompt?

A structured request sent to a chat model containing messages and potentially model/request options.

Question 5 — What is ChatClient?

A higher-level fluent API that makes constructing and executing chat-model requests easier.

Question 6 — What is the difference between ChatClient and ChatModel?
ChatClient → Higher-level fluent API

ChatModel  → Core model abstraction

Question 7 — What is the difference between Prompt and Message?
Message → Individual conversational unit

Prompt  → Complete structured model request

Question 8 — What is the difference between ChatModel and EmbeddingModel?
ChatModel
    ↓
Generate response

EmbeddingModel
    ↓
Generate vector representation

34. Hands-On Study Plan

The roadmap gives this topic approximately 3–4 hours.

Don't spend the entire time reading.

Hour 1 — Understand the Architecture

Draw this diagram yourself:

Application
    |
    v
ChatClient
    |
    v
Prompt
    |
    v
Messages
    |
    v
ChatModel
    |
    v
Provider
    |
    v
LLM


Then explain every arrow in your own words.

Hour 2 — Build a Simple Chat API

Build:

GET /api/ask?question=...
            |
            v
       Controller
            |
            v
         Service
            |
            v
       ChatClient
            |
            v
           LLM
            |
            v
        String response


Experiment with:

.user("Explain dependency injection.")


Then add:

.system("You are an expert Java instructor.")


Observe how the system instruction affects the response.

Hour 3 — Understand the Lower-Level Abstraction

Experiment with:

Prompt
   |
   v
ChatModel
   |
   v
ChatResponse


Compare it with:

ChatClient
   |
   v
Prompt
   |
   v
ChatModel


Your goal is to understand what ChatClient provides on top of ChatModel.

Hour 4 — Embeddings

Create embeddings for several sentences:

"Java is a programming language."

"Spring Boot is a Java framework."

"The weather is hot today."


Then compare their similarity.

This prepares you for:

Phase 7 — Embeddings

35. Mini Exercise

Design this API without looking at the answer:

POST /api/explain

{
    "topic": "Dependency Injection"
}


Your architecture should look like:

HTTP Request
      |
      v
Controller
      |
      v
Service
      |
      v
ChatClient
      |
      v
Prompt
      |
      +---- System Message
      |
      +---- User Message
      |
      v
ChatModel
      |
      v
LLM
      |
      v
ChatResponse
      |
      v
String
      |
      v
HTTP Response


Then answer:

Why do we need ChatClient?
Why does ChatModel exist?
Why isn't a prompt simply a string?
Why are messages separate objects?
When would you want ChatResponse instead of String?
Why is EmbeddingModel separate from ChatModel?

If you can answer all six clearly, you understand the core abstractions.

36. Final Cheat Sheet
Concept	Mental Model
ChatClient	Easy way to build and execute AI requests
ChatModel	Abstraction over a chat/generative model
Prompt	Complete structured model request
Message	Individual conversational unit
SystemMessage	Instructions/context for the model
UserMessage	User input
AssistantMessage	Model/assistant response
ChatResponse	Structured model response
EmbeddingModel	Input → vector
37. One-Line Summary
ChatClient
    ↓
Prompt
    ↓
Messages
    ↓
ChatModel
    ↓
Provider
    ↓
LLM


And for embeddings:

Text
    ↓
EmbeddingModel
    ↓
Vector
    ↓
Vector Store


Key takeaway: Spring AI provides abstractions around AI models, prompts, messages, responses, and embeddings so that your Spring application can work with AI capabilities without being tightly coupled to the implementation details of a particular AI provider.

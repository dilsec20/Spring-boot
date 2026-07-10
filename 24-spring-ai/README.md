# 🤖 Spring AI — Complete In-Depth Guide

> **"Spring AI brings AI capabilities to Spring Boot. Integrate LLMs, embeddings, vector stores, and RAG patterns with the familiar Spring programming model."**

---

## 📑 Table of Contents

1. [What is Spring AI?](#1-what-is-spring-ai)
2. [Setup & Configuration](#2-setup--configuration)
3. [Chat Models (LLM Integration)](#3-chat-models-llm-integration)
4. [Prompt Engineering](#4-prompt-engineering)
5. [Structured Output](#5-structured-output)
6. [Embeddings & Vector Stores](#6-embeddings--vector-stores)
7. [RAG (Retrieval-Augmented Generation)](#7-rag-retrieval-augmented-generation)
8. [Function Calling (Tool Use)](#8-function-calling-tool-use)
9. [Image Generation](#9-image-generation)
10. [Multi-Model Support](#10-multi-model-support)
11. [Building an AI Chat Application](#11-building-an-ai-chat-application)
12. [Best Practices](#12-best-practices)
13. [Interview Questions & Answers (50+)](#13-interview-questions--answers-50)

---

## 1. What is Spring AI?

```
Spring AI = Spring Boot + AI/ML Integration

What it provides:
  ✅ Unified API across LLM providers (OpenAI, Anthropic, Google, Ollama)
  ✅ Chat completions, embeddings, image generation
  ✅ Prompt templates (like Thymeleaf for prompts)
  ✅ Structured output (JSON from LLM responses)
  ✅ Vector stores (for RAG / semantic search)
  ✅ Function calling (LLM can call your Java methods)
  ✅ Spring ecosystem integration (auto-config, DI, testing)
```

---

## 2. Setup & Configuration

```xml
<!-- Spring AI BOM -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>1.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- OpenAI integration -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
</dependency>

<!-- For Ollama (local models) -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
</dependency>
```

```properties
# OpenAI
spring.ai.openai.api-key=${OPENAI_API_KEY}
spring.ai.openai.chat.options.model=gpt-4o
spring.ai.openai.chat.options.temperature=0.7

# Ollama (local)
spring.ai.ollama.base-url=http://localhost:11434
spring.ai.ollama.chat.options.model=llama3
```

---

## 3. Chat Models (LLM Integration)

```java
@RestController
@RequestMapping("/api/ai")
@RequiredArgsConstructor
public class AiController {
    
    private final ChatModel chatModel;  // Auto-injected by Spring AI
    
    // Simple chat
    @GetMapping("/chat")
    public String chat(@RequestParam String message) {
        return chatModel.call(message);  // Send to LLM, get response
    }
    
    // Chat with options
    @PostMapping("/chat")
    public String chatWithOptions(@RequestBody ChatRequest request) {
        Prompt prompt = new Prompt(
            request.getMessage(),
            OpenAiChatOptions.builder()
                .model("gpt-4o")
                .temperature(0.7)
                .maxTokens(500)
                .build()
        );
        
        ChatResponse response = chatModel.call(prompt);
        return response.getResult().getOutput().getText();
    }
    
    // Streaming response
    @GetMapping(value = "/chat/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> chatStream(@RequestParam String message) {
        Prompt prompt = new Prompt(message);
        return chatModel.stream(prompt)
            .map(response -> response.getResult().getOutput().getText());
    }
    
    // Conversation with history
    @PostMapping("/conversation")
    public String conversation(@RequestBody List<MessageDTO> messages) {
        List<Message> chatMessages = messages.stream()
            .map(m -> switch (m.getRole()) {
                case "user" -> new UserMessage(m.getContent());
                case "assistant" -> new AssistantMessage(m.getContent());
                case "system" -> new SystemMessage(m.getContent());
                default -> throw new IllegalArgumentException("Unknown role");
            })
            .toList();
        
        Prompt prompt = new Prompt(chatMessages);
        return chatModel.call(prompt).getResult().getOutput().getText();
    }
}
```

---

## 4. Prompt Engineering

```java
@Service
public class PromptService {
    
    private final ChatModel chatModel;
    
    // System prompt (sets AI behavior)
    public String chatWithPersona(String userMessage) {
        SystemMessage system = new SystemMessage("""
            You are a friendly Java expert. Answer questions about Spring Boot 
            with code examples. Keep responses concise and practical.
            Always mention best practices.
        """);
        
        UserMessage user = new UserMessage(userMessage);
        
        Prompt prompt = new Prompt(List.of(system, user));
        return chatModel.call(prompt).getResult().getOutput().getText();
    }
    
    // Prompt template (reusable with variables)
    public String generateEmail(String recipientName, String topic) {
        PromptTemplate template = new PromptTemplate("""
            Write a professional email to {name} about {topic}.
            Keep it brief and friendly. Include a subject line.
        """);
        
        Map<String, Object> variables = Map.of(
            "name", recipientName,
            "topic", topic
        );
        
        Prompt prompt = template.create(variables);
        return chatModel.call(prompt).getResult().getOutput().getText();
    }
    
    // Few-shot prompting
    public String classify(String text) {
        String prompt = """
            Classify the following text as POSITIVE, NEGATIVE, or NEUTRAL.
            
            Examples:
            "I love this product!" → POSITIVE
            "This is terrible." → NEGATIVE
            "The package arrived today." → NEUTRAL
            
            Text: "%s"
            Classification:
        """.formatted(text);
        
        return chatModel.call(prompt).trim();
    }
}
```

---

## 5. Structured Output

```java
// Get JSON objects from LLM responses (type-safe!)

public record MovieRecommendation(
    String title,
    int year,
    String genre,
    String reason
) {}

@Service
public class MovieService {
    
    private final ChatModel chatModel;
    
    public List<MovieRecommendation> recommendMovies(String genre) {
        BeanOutputConverter<List<MovieRecommendation>> converter = 
            new BeanOutputConverter<>(new ParameterizedTypeReference<List<MovieRecommendation>>() {});
        
        String prompt = """
            Recommend 3 %s movies. 
            %s
        """.formatted(genre, converter.getFormat());
        
        String response = chatModel.call(prompt);
        return converter.convert(response);  // Parsed to Java objects!
    }
}
```

---

## 6. Embeddings & Vector Stores

```java
// Embeddings = Convert text to numerical vectors
// Vector Store = Database for storing and searching vectors

@Service
public class EmbeddingService {
    
    private final EmbeddingModel embeddingModel;
    private final VectorStore vectorStore;
    
    // Generate embedding for text
    public float[] getEmbedding(String text) {
        EmbeddingResponse response = embeddingModel.call(
            new EmbeddingRequest(List.of(text), EmbeddingOptions.EMPTY)
        );
        return response.getResult().getOutput();
    }
    
    // Store documents in vector store
    public void storeDocuments(List<String> texts) {
        List<Document> documents = texts.stream()
            .map(text -> new Document(text))
            .toList();
        
        vectorStore.add(documents);
    }
    
    // Semantic search (find similar documents)
    public List<Document> search(String query) {
        return vectorStore.similaritySearch(
            SearchRequest.builder()
                .query(query)
                .topK(5)          // Top 5 results
                .similarityThreshold(0.7)  // Minimum similarity
                .build()
        );
    }
}
```

---

## 7. RAG (Retrieval-Augmented Generation)

```
RAG Flow:
1. User asks a question
2. Search vector store for relevant documents
3. Include retrieved documents in the prompt
4. LLM generates answer based on YOUR data

Why RAG?
  LLM alone: "I don't know about your company's internal policies"
  LLM + RAG: "Based on your HR policy document, the vacation policy is..."
```

```java
@Service
public class RagService {
    
    private final ChatModel chatModel;
    private final VectorStore vectorStore;
    
    public String askWithContext(String question) {
        // Step 1: Search for relevant documents
        List<Document> relevantDocs = vectorStore.similaritySearch(
            SearchRequest.builder().query(question).topK(3).build()
        );
        
        // Step 2: Build context from retrieved documents
        String context = relevantDocs.stream()
            .map(Document::getText)
            .collect(Collectors.joining("\n\n"));
        
        // Step 3: Create prompt with context
        String prompt = """
            Answer the question based ONLY on the following context.
            If the answer is not in the context, say "I don't have that information."
            
            Context:
            %s
            
            Question: %s
            
            Answer:
        """.formatted(context, question);
        
        // Step 4: Get AI response
        return chatModel.call(prompt);
    }
}
```

---

## 8. Function Calling (Tool Use)

```java
// LLM can call YOUR Java methods!

@Configuration
public class AiFunctions {
    
    @Bean
    @Description("Get current weather for a city")
    public Function<WeatherRequest, WeatherResponse> getWeather() {
        return request -> {
            // Call weather API
            return new WeatherResponse(request.city(), 25, "Sunny");
        };
    }
    
    @Bean
    @Description("Search for products in the database")
    public Function<ProductSearchRequest, List<Product>> searchProducts(ProductService productService) {
        return request -> productService.search(request.query());
    }
}

record WeatherRequest(String city) {}
record WeatherResponse(String city, int temperature, String condition) {}
record ProductSearchRequest(String query) {}

// Usage:
@GetMapping("/chat")
public String chat(@RequestParam String message) {
    Prompt prompt = new Prompt(message,
        OpenAiChatOptions.builder()
            .functions(Set.of("getWeather", "searchProducts"))
            .build());
    
    return chatModel.call(prompt).getResult().getOutput().getText();
}
// User: "What's the weather in Mumbai?"
// AI calls getWeather("Mumbai") → gets data → responds naturally
```

---

## 9. Image Generation

```java
@Service
public class ImageService {
    
    private final ImageModel imageModel;
    
    public String generateImage(String description) {
        ImagePrompt prompt = new ImagePrompt(description,
            OpenAiImageOptions.builder()
                .model("dall-e-3")
                .quality("standard")
                .width(1024)
                .height(1024)
                .build());
        
        ImageResponse response = imageModel.call(prompt);
        return response.getResult().getOutput().getUrl();  // Image URL
    }
}
```

---

## 10. Multi-Model Support

```
Spring AI supports:

Cloud Models:
  ✅ OpenAI (GPT-4, GPT-4o)
  ✅ Anthropic (Claude)
  ✅ Google (Gemini)
  ✅ AWS Bedrock (multiple models)
  ✅ Azure OpenAI
  ✅ Mistral AI

Local Models:
  ✅ Ollama (Llama, Mistral, DeepSeek, etc.)

Vector Stores:
  ✅ PostgreSQL/PGVector
  ✅ Redis
  ✅ Pinecone
  ✅ Chroma
  ✅ Milvus

Switch provider = change dependency + properties. Code stays the same!
```

---

## 11. Building an AI Chat Application

```java
@RestController
@RequestMapping("/api/chat")
public class ChatAppController {
    
    private final ChatModel chatModel;
    private final VectorStore vectorStore;
    
    // Chat with memory and RAG
    @PostMapping
    public ChatResponseDTO chat(@RequestBody ChatRequestDTO request) {
        
        // System prompt
        SystemMessage system = new SystemMessage("""
            You are a helpful assistant for our e-commerce platform.
            Be friendly, concise, and helpful.
        """);
        
        // RAG: Search for relevant product info
        List<Document> context = vectorStore.similaritySearch(
            SearchRequest.builder().query(request.getMessage()).topK(3).build());
        
        String contextText = context.stream()
            .map(Document::getText).collect(Collectors.joining("\n"));
        
        UserMessage user = new UserMessage("""
            Context from our product catalog:
            %s
            
            User question: %s
        """.formatted(contextText, request.getMessage()));
        
        // Previous conversation history
        List<Message> messages = new ArrayList<>();
        messages.add(system);
        for (var msg : request.getHistory()) {
            messages.add(msg.getRole().equals("user") 
                ? new UserMessage(msg.getContent())
                : new AssistantMessage(msg.getContent()));
        }
        messages.add(user);
        
        String response = chatModel.call(new Prompt(messages))
            .getResult().getOutput().getText();
        
        return new ChatResponseDTO(response);
    }
}
```

---

## 12. Best Practices

1. **Use system prompts** — Define AI behavior and constraints
2. **Implement RAG** — Ground responses in your data
3. **Stream long responses** — Better UX with SSE
4. **Set temperature appropriately** — 0 for factual, 0.7 for creative
5. **Validate AI output** — Don't trust LLM responses blindly
6. **Cache responses** — Same question = same answer (save tokens)
7. **Set max tokens** — Prevent runaway costs
8. **Use structured output** — Type-safe parsing of LLM responses
9. **Handle rate limits** — Retry with backoff
10. **Monitor token usage** — Track costs per endpoint

---

## 13. Interview Questions & Answers (50+)

### Beginner

**Q1: What is Spring AI?** Spring framework integration for AI/ML. Provides unified API for LLMs, embeddings, vector stores with Spring conventions.

**Q2: What is an LLM?** Large Language Model — AI trained on text data to understand and generate human language. GPT-4, Claude, Llama.

**Q3: What is ChatModel?** Spring AI interface for chat completions. Sends prompts, receives AI-generated text.

**Q4: What is a prompt?** Input text sent to the LLM. Can include system instructions, user messages, and context.

**Q5: What is temperature?** Controls randomness. 0 = deterministic. 1 = creative. 0.7 = balanced.

**Q6: What is a token?** Unit of text for LLMs. ~4 characters = 1 token. Used for pricing and limits.

**Q7: What is streaming?** Receiving LLM response word-by-word as it's generated, instead of waiting for complete response.

**Q8: OpenAI vs Ollama?** OpenAI: cloud API (GPT-4). Ollama: run models locally (Llama, Mistral, DeepSeek).

---

### Intermediate

**Q9: What is RAG?** Retrieval-Augmented Generation. Search your documents, include them in the prompt, LLM answers based on your data.

**Q10: What is an embedding?** Numerical vector representation of text. Similar texts have similar vectors. Used for semantic search.

**Q11: What is a vector store?** Database optimized for storing and searching vector embeddings. PGVector, Pinecone, Chroma.

**Q12: What is function calling?** LLM can call your Java methods. LLM decides when to call, provides arguments, you execute and return results.

**Q13: What is structured output?** Parsing LLM text response into Java objects (JSON → POJO). BeanOutputConverter in Spring AI.

**Q14: What is a system prompt?** Instructions that define AI behavior, role, and constraints. Set once per conversation.

**Q15: What is prompt engineering?** Crafting effective prompts to get desired AI outputs. Includes few-shot, chain-of-thought, structured instructions.

---

### Rapid-Fire (Q16–Q50)

**Q16: Spring AI model abstraction?** ChatModel, EmbeddingModel, ImageModel — unified across providers.

**Q17: How to switch from OpenAI to Ollama?** Change dependency + properties. Code stays same (interface-based).

**Q18: What is few-shot prompting?** Providing examples in the prompt to guide LLM behavior.

**Q19: What is chain-of-thought?** Asking LLM to think step-by-step for better reasoning.

**Q20: What is max_tokens?** Maximum response length. Limits cost and response size.

**Q21: What is a Document in Spring AI?** Text content + metadata stored in vector store.

**Q22: What is similarity search?** Finding documents with vectors closest to query vector. Cosine similarity.

**Q23: What is top_k?** Number of most similar results to return from vector search.

**Q24: What is similarity threshold?** Minimum similarity score to include in results.

**Q25: What is PGVector?** PostgreSQL extension for vector storage and similarity search.

**Q26: What is Pinecone?** Cloud-native vector database as a service.

**Q27: What is hallucination?** LLM generating false or made-up information. RAG helps reduce this.

**Q28: PromptTemplate purpose?** Reusable prompt with variables. Like Thymeleaf for prompts.

**Q29: What is ChatResponse?** Response object containing generated text, metadata, usage stats.

**Q30: What is Flux in streaming?** Reactive type for streaming responses (Project Reactor).

**Q31: How to handle API rate limits?** Retry with exponential backoff. Spring Retry or Resilience4j.

**Q32: Cost of LLM API calls?** Based on tokens. GPT-4: ~$0.03/1K input tokens, ~$0.06/1K output tokens.

**Q33: Local vs cloud models?** Local: free, private, slower. Cloud: paid, faster, more capable.

**Q34: What is fine-tuning?** Training a model on your specific data for better domain performance.

**Q35: What is context window?** Maximum input + output tokens. GPT-4: 128K tokens.

**Q36: What is multi-modal AI?** AI that handles text, images, audio, video. GPT-4o, Gemini.

**Q37: What is an AI agent?** AI that can reason, plan, and use tools to accomplish tasks autonomously.

**Q38: What is Spring AI Advisor?** Interceptor pattern for prompts. Add logging, RAG, memory around AI calls.

**Q39: What is conversation memory?** Storing chat history so AI maintains context across messages.

**Q40: What is AI safety?** Preventing harmful, biased, or unsafe AI outputs. Content filtering, guardrails.

**Q41: What is model evaluation?** Measuring AI response quality. Accuracy, relevance, groundedness.

**Q42: How to test AI features?** Mock ChatModel in unit tests. Integration tests with local Ollama.

**Q43: What is prompt injection?** User manipulates prompt to override system instructions. Security concern.

**Q44: Embeddings dimension?** Number of dimensions in vector. OpenAI: 1536-3072. Higher = more info.

**Q45: What is chunking in RAG?** Splitting large documents into smaller pieces for embedding. 500-1000 tokens per chunk.

**Q46: What is overlap in chunking?** Shared text between chunks to maintain context at boundaries.

**Q47: Spring AI vs LangChain?** Spring AI: Java/Spring native. LangChain: Python/JS. Same concepts, different ecosystems.

**Q48: What is Ollama?** Tool to run LLMs locally. Download and run: `ollama run llama3`.

**Q49: What is Hugging Face?** Platform for ML models, datasets, and demos. Model hub.

**Q50: Future of Spring AI?** Agents, multi-modal, tool use, evaluation frameworks. Rapidly evolving.

---

## 📚 References

- [Spring AI Documentation](https://docs.spring.io/spring-ai/reference/)
- [Spring AI GitHub](https://github.com/spring-projects/spring-ai)
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference)
- [Ollama](https://ollama.ai/)

---

> **Previous Topic:** [← 23 - Cloud Deployment](../23-cloud-deployment/README.md)  
> **Next Topic:** [25 - DeepSeek + Ollama →](../25-deepseek-ollama-spring/README.md)

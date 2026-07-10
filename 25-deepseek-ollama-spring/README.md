# 🧠 DeepSeek + Ollama + Spring Boot — Complete In-Depth Guide

> **"Run powerful AI models locally for FREE with Ollama + DeepSeek. No API keys, no cloud costs, complete privacy. Integrate seamlessly with Spring Boot."**

---

## 📑 Table of Contents

1. [What is DeepSeek?](#1-what-is-deepseek)
2. [What is Ollama?](#2-what-is-ollama)
3. [Installing Ollama & Models](#3-installing-ollama--models)
4. [Running DeepSeek Locally](#4-running-deepseek-locally)
5. [Spring Boot + Ollama Integration](#5-spring-boot--ollama-integration)
6. [Building a Chat API](#6-building-a-chat-api)
7. [Code Generation with DeepSeek Coder](#7-code-generation-with-deepseek-coder)
8. [RAG with Local Models](#8-rag-with-local-models)
9. [Local vs Cloud Models](#9-local-vs-cloud-models)
10. [Production Deployment](#10-production-deployment)
11. [Other Ollama Models](#11-other-ollama-models)
12. [Interview Questions & Answers (50+)](#12-interview-questions--answers-50)

---

## 1. What is DeepSeek?

```
DeepSeek is an open-source LLM family from DeepSeek AI (China):

DeepSeek Models:
┌─────────────────────────────────────────────┐
│ DeepSeek-V3        │ General purpose (671B) │
│ DeepSeek-R1        │ Reasoning model        │
│ DeepSeek-Coder-V2  │ Code generation        │
├─────────────────────────────────────────────┤
│ Key Features:                               │
│ ✅ Open-source (MIT license)               │
│ ✅ Competitive with GPT-4                  │
│ ✅ Run locally with Ollama                 │
│ ✅ Free to use (no API costs)              │
│ ✅ Full data privacy                       │
└─────────────────────────────────────────────┘
```

---

## 2. What is Ollama?

```
Ollama = "Docker for LLMs"

One command to run any open-source LLM locally:
  ollama run deepseek-r1
  ollama run llama3
  ollama run mistral
  ollama run codellama

Features:
  ✅ Simple CLI (ollama run model-name)
  ✅ REST API (localhost:11434)
  ✅ GPU acceleration (NVIDIA, Apple Silicon)
  ✅ Model management (pull, list, remove)
  ✅ Modelfile (customize models)
  ✅ Spring AI integration
```

---

## 3. Installing Ollama & Models

```bash
# ═══ Install Ollama ═══
# Windows: Download from https://ollama.ai/download
# macOS:   brew install ollama
# Linux:   curl -fsSL https://ollama.ai/install.sh | sh

# Verify installation
ollama --version

# ═══ Pull Models ═══
ollama pull deepseek-r1          # Reasoning model (~4.7GB for 7B)
ollama pull deepseek-r1:14b      # Larger, more capable
ollama pull deepseek-coder-v2    # Code generation
ollama pull llama3:8b            # Meta's Llama 3
ollama pull mistral              # Mistral AI
ollama pull nomic-embed-text     # Embedding model (for RAG)

# ═══ List Models ═══
ollama list

# ═══ Run Model (Interactive) ═══
ollama run deepseek-r1
>>> What is Spring Boot?
# AI responds in your terminal!

# ═══ Serve API ═══
ollama serve
# API available at http://localhost:11434

# ═══ Test API ═══
curl http://localhost:11434/api/chat -d '{
  "model": "deepseek-r1",
  "messages": [{"role": "user", "content": "Hello!"}],
  "stream": false
}'
```

### Hardware Requirements

| Model Size | RAM Required | GPU VRAM | Quality |
|------------|-------------|----------|---------|
| 1.5B | 4 GB | 2 GB | Basic |
| 7B | 8 GB | 6 GB | Good |
| 14B | 16 GB | 12 GB | Great |
| 32B | 32 GB | 24 GB | Excellent |
| 70B | 64 GB | 48 GB | Near GPT-4 |

---

## 4. Running DeepSeek Locally

```bash
# Start Ollama server
ollama serve

# In another terminal, run DeepSeek
ollama run deepseek-r1

# Chat!
>>> Explain dependency injection in Spring Boot with a code example
```

### Ollama REST API

```bash
# Chat completion
curl http://localhost:11434/api/chat -d '{
  "model": "deepseek-r1",
  "messages": [
    {"role": "system", "content": "You are a Java expert."},
    {"role": "user", "content": "Write a Spring Boot REST controller"}
  ],
  "stream": false
}'

# Generate embeddings
curl http://localhost:11434/api/embeddings -d '{
  "model": "nomic-embed-text",
  "prompt": "Spring Boot is a Java framework"
}'

# List models
curl http://localhost:11434/api/tags
```

---

## 5. Spring Boot + Ollama Integration

### Dependencies

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
</dependency>
```

### Configuration

```properties
# application.properties
spring.ai.ollama.base-url=http://localhost:11434
spring.ai.ollama.chat.options.model=deepseek-r1
spring.ai.ollama.chat.options.temperature=0.7
spring.ai.ollama.chat.options.num-predict=2048

# For embeddings
spring.ai.ollama.embedding.options.model=nomic-embed-text
```

### Basic Usage

```java
@RestController
@RequestMapping("/api/ai")
@RequiredArgsConstructor
public class DeepSeekController {
    
    private final ChatModel chatModel;  // Ollama ChatModel auto-configured
    
    @GetMapping("/chat")
    public String chat(@RequestParam String question) {
        return chatModel.call(question);
    }
    
    @PostMapping("/chat")
    public Map<String, String> chatWithSystem(@RequestBody ChatRequestDTO request) {
        SystemMessage system = new SystemMessage("""
            You are a senior Java developer. Provide concise, practical answers 
            with code examples. Use Spring Boot best practices.
        """);
        
        UserMessage user = new UserMessage(request.getMessage());
        
        Prompt prompt = new Prompt(List.of(system, user),
            OllamaOptions.builder()
                .model("deepseek-r1")
                .temperature(0.7)
                .build());
        
        String response = chatModel.call(prompt)
            .getResult().getOutput().getText();
        
        return Map.of("response", response);
    }
    
    // Streaming response
    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> stream(@RequestParam String question) {
        return chatModel.stream(new Prompt(question))
            .map(r -> r.getResult().getOutput().getText())
            .filter(text -> text != null && !text.isEmpty());
    }
}
```

---

## 6. Building a Chat API

```java
@RestController
@RequestMapping("/api/chat")
@RequiredArgsConstructor
public class ChatController {
    
    private final ChatModel chatModel;
    
    // Conversation with history
    @PostMapping
    public ResponseEntity<ChatResponseDTO> chat(@RequestBody ConversationRequest request) {
        
        List<Message> messages = new ArrayList<>();
        
        // System prompt
        messages.add(new SystemMessage("""
            You are an AI assistant for a Spring Boot learning platform.
            Help users understand Java and Spring Boot concepts.
            Provide code examples when relevant.
            Be encouraging and patient with beginners.
        """));
        
        // Add conversation history
        for (var msg : request.getHistory()) {
            if ("user".equals(msg.getRole())) {
                messages.add(new UserMessage(msg.getContent()));
            } else {
                messages.add(new AssistantMessage(msg.getContent()));
            }
        }
        
        // Current message
        messages.add(new UserMessage(request.getMessage()));
        
        Prompt prompt = new Prompt(messages);
        String response = chatModel.call(prompt).getResult().getOutput().getText();
        
        return ResponseEntity.ok(new ChatResponseDTO(response));
    }
}

// DTOs
record ConversationRequest(String message, List<MessageDTO> history) {}
record MessageDTO(String role, String content) {}
record ChatResponseDTO(String response) {}
```

---

## 7. Code Generation with DeepSeek Coder

```java
@Service
@RequiredArgsConstructor
public class CodeGenService {
    
    private final ChatModel chatModel;
    
    public String generateCode(String description, String language) {
        String prompt = """
            Generate %s code for the following requirement:
            %s
            
            Requirements:
            - Clean, production-ready code
            - Include comments explaining the logic
            - Follow %s best practices
            - Handle edge cases and errors
            
            Return ONLY the code, no explanations.
        """.formatted(language, description, language);
        
        return chatModel.call(new Prompt(prompt,
            OllamaOptions.builder()
                .model("deepseek-coder-v2")
                .temperature(0.2)  // Low temperature for code
                .build()))
            .getResult().getOutput().getText();
    }
    
    public String reviewCode(String code) {
        String prompt = """
            Review the following code for:
            1. Bugs and potential issues
            2. Performance improvements
            3. Security vulnerabilities
            4. Code style and best practices
            
            Code:
            ```
            %s
            ```
            
            Provide specific, actionable feedback.
        """.formatted(code);
        
        return chatModel.call(prompt);
    }
    
    public String explainCode(String code) {
        return chatModel.call("""
            Explain the following code line by line for a beginner:
            ```
            %s
            ```
        """.formatted(code));
    }
}
```

---

## 8. RAG with Local Models

```java
@Service
@RequiredArgsConstructor
public class LocalRagService {
    
    private final ChatModel chatModel;           // DeepSeek via Ollama
    private final EmbeddingModel embeddingModel;  // nomic-embed-text via Ollama
    private final VectorStore vectorStore;         // PGVector
    
    // Index documents
    public void indexDocuments(List<String> texts) {
        List<Document> documents = texts.stream()
            .map(text -> new Document(text))
            .toList();
        vectorStore.add(documents);
    }
    
    // RAG query — completely local, completely free!
    public String ask(String question) {
        // 1. Semantic search in vector store
        List<Document> relevant = vectorStore.similaritySearch(
            SearchRequest.builder().query(question).topK(3).build());
        
        String context = relevant.stream()
            .map(Document::getText)
            .collect(Collectors.joining("\n---\n"));
        
        // 2. Generate answer with context
        String prompt = """
            Based on the following context, answer the question.
            If the answer is not in the context, say so.
            
            Context:
            %s
            
            Question: %s
        """.formatted(context, question);
        
        return chatModel.call(prompt);
    }
}
```

---

## 9. Local vs Cloud Models

| Feature | Local (Ollama) | Cloud (OpenAI) |
|---------|---------------|----------------|
| Cost | FREE ✅ | Pay per token 💰 |
| Privacy | 100% private ✅ | Data sent to cloud |
| Speed | Depends on hardware | Fast (powerful GPUs) |
| Quality (7B) | Good | N/A |
| Quality (70B) | Excellent | N/A |
| Quality (GPT-4) | N/A | Best |
| Internet | Not required | Required |
| Setup | Install Ollama | Get API key |
| Scaling | Limited by hardware | Unlimited |

---

## 10. Production Deployment

```yaml
# Docker Compose: Spring Boot + Ollama + PGVector
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_AI_OLLAMA_BASE_URL=http://ollama:11434
      - SPRING_DATASOURCE_URL=jdbc:postgresql://pgvector:5432/ragdb
    depends_on:
      - ollama
      - pgvector

  ollama:
    image: ollama/ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama-data:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]  # GPU support

  pgvector:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_DB: ragdb
      POSTGRES_PASSWORD: password
    volumes:
      - pgvector-data:/var/lib/postgresql/data

volumes:
  ollama-data:
  pgvector-data:
```

---

## 11. Other Ollama Models

| Model | Size | Best For |
|-------|------|----------|
| `deepseek-r1` | 1.5B-671B | Reasoning, general tasks |
| `deepseek-coder-v2` | 16B-236B | Code generation |
| `llama3` | 8B/70B | General purpose |
| `mistral` | 7B | Fast, efficient |
| `codellama` | 7B/34B | Code generation |
| `phi3` | 3.8B | Small, fast |
| `gemma2` | 9B/27B | Google's model |
| `qwen2.5` | 7B/72B | Multilingual |
| `nomic-embed-text` | - | Embeddings (for RAG) |

---

## 12. Interview Questions & Answers (50+)

### Beginner

**Q1: What is DeepSeek?** Open-source LLM family. Competitive with GPT-4. Models include DeepSeek-V3, R1 (reasoning), Coder.

**Q2: What is Ollama?** Tool to run open-source LLMs locally. One command: `ollama run llama3`. REST API on localhost:11434.

**Q3: Can I run AI locally for free?** Yes! Ollama + open-source models (Llama, DeepSeek, Mistral). No API keys, no cloud costs.

**Q4: What hardware do I need?** 7B model: 8GB RAM. 14B: 16GB. 70B: 64GB. GPU recommended but not required.

**Q5: How does Spring AI connect to Ollama?** `spring-ai-ollama-spring-boot-starter`. Auto-configures ChatModel. Set `spring.ai.ollama.base-url`.

**Q6: DeepSeek-R1 vs DeepSeek-Coder?** R1: general reasoning. Coder: optimized for code generation, debugging, explanation.

**Q7: What is a quantized model?** Compressed model using less precision (4-bit instead of 16-bit). Smaller, faster, slightly lower quality.

**Q8: What is GGUF format?** File format for quantized LLMs. Used by llama.cpp and Ollama.

---

### Intermediate

**Q9: How does Ollama serve models?** Loads model into memory (RAM/VRAM), starts HTTP server on 11434, processes requests sequentially.

**Q10: What is a Modelfile?** Ollama's Dockerfile equivalent. Customize model with system prompts, parameters, stop tokens.

**Q11: How to use embeddings locally?** Use `nomic-embed-text` model in Ollama. Spring AI auto-configures EmbeddingModel.

**Q12: How to implement RAG locally?** Ollama (chat + embeddings) + PGVector (vector store). All running on your machine.

**Q13: Can I fine-tune with Ollama?** Ollama doesn't support fine-tuning directly. Use PEFT/LoRA tools, then import the model.

**Q14: GPU vs CPU for LLMs?** GPU: 10-50x faster inference. CPU: slower but works. Apple Silicon (M1+): excellent for local LLMs.

**Q15: How to stream responses?** Use `chatModel.stream()` returning `Flux<ChatResponse>`. Send as Server-Sent Events.

---

### Rapid-Fire (Q16–Q50)

**Q16: Default Ollama port?** 11434.

**Q17: `ollama pull model`?** Downloads model to local machine.

**Q18: `ollama list`?** Shows all downloaded models.

**Q19: `ollama rm model`?** Removes a model.

**Q20: `ollama show model`?** Shows model details (size, parameters, license).

**Q21: What is context length?** Maximum input+output tokens. Varies by model: 4K, 8K, 32K, 128K.

**Q22: What is num_predict?** Maximum tokens to generate in response. Like max_tokens.

**Q23: What is top_p?** Nucleus sampling. Alternative to temperature for controlling randomness.

**Q24: What is repeat_penalty?** Penalizes repeated words. Higher = less repetition.

**Q25: Can multiple users share one Ollama?** Yes, but requests are processed sequentially. Use queue for production.

**Q26: Ollama Docker image?** `ollama/ollama`. Run: `docker run -d -p 11434:11434 ollama/ollama`.

**Q27: GPU in Docker for Ollama?** Use `--gpus all` flag or Docker Compose `deploy.resources.reservations.devices`.

**Q28: Can Ollama run on Windows?** Yes, native Windows installer available.

**Q29: What is llama.cpp?** C++ library for running LLMs on CPU. Ollama uses it internally.

**Q30: Open-source vs proprietary models?** Open: free, customizable, run locally. Proprietary: better quality (GPT-4), cloud only.

**Q31: DeepSeek license?** MIT license — fully open for commercial use.

**Q32: What is MoE (Mixture of Experts)?** Architecture where only subset of parameters activate per input. DeepSeek-V3 uses MoE.

**Q33: What is distillation?** Training smaller model to mimic larger model. Cheaper inference with similar quality.

**Q34: Model size vs quality tradeoff?** Larger = better quality but more resources. 7B for experiments, 70B for production quality.

**Q35: Can I use Ollama offline?** Yes! After pulling the model, no internet needed.

**Q36: Ollama vs vLLM?** Ollama: simple, local use. vLLM: high-throughput serving, production inference engine.

**Q37: How to monitor Ollama?** Check `/api/tags` for loaded models. System `top`/`nvidia-smi` for resource usage.

**Q38: What is inference?** Using a trained model to generate predictions/text. Not training.

**Q39: What is prompt caching?** Reusing computed attention for repeated prompt prefixes. Faster subsequent calls.

**Q40: How to compare model quality?** Run same prompts, compare outputs. Use benchmarks (MMLU, HumanEval, GSM8K).

**Q41: What is token/second (t/s)?** Speed metric. 7B on M2: ~30 t/s. 70B on A100: ~40 t/s.

**Q42: What is batch inference?** Processing multiple prompts simultaneously. Better GPU utilization.

**Q43: Can Ollama do vision?** Yes, with multimodal models like `llava`. Send images + text.

**Q44: What is system prompt in Modelfile?** Default system instruction baked into the model configuration.

**Q45: How to create custom model?** Write a Modelfile, `ollama create mymodel -f Modelfile`.

**Q46: What is GPTQ vs GGUF?** GPTQ: GPU-optimized quantization. GGUF: CPU+GPU flexible format.

**Q47: Spring AI OllamaOptions?** Configure model, temperature, num_predict, top_p per request.

**Q48: Health check for Ollama?** `curl http://localhost:11434` returns "Ollama is running".

**Q49: How to update models?** `ollama pull model:tag` re-downloads latest version.

**Q50: Future of local AI?** Smaller, faster models. On-device AI. Privacy-first applications.

---

## 📚 References

- [Ollama Official](https://ollama.ai/)
- [DeepSeek AI](https://www.deepseek.com/)
- [Spring AI Ollama Docs](https://docs.spring.io/spring-ai/reference/api/chat/ollama-chat.html)
- [Ollama Model Library](https://ollama.ai/library)
- [Ollama GitHub](https://github.com/ollama/ollama)

---

> **Previous Topic:** [← 24 - Spring AI](../24-spring-ai/README.md)  
> **Next Topic:** [26 - Microservices →](../26-microservices/README.md)

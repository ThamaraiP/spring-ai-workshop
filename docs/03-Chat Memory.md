# Introduce Chat Memory

## Objective
Add in-memory conversation history for contextual responses.

## pom.xml additions
```xml
<dependency>
  <groupId>org.springframework.ai</groupId>
  <artifactId>spring-ai-starter-model-chat-memory</artifactId>
</dependency>
```

## Config
```java
package com.example.spring_ai_demo.config;
import org.springframework.ai.chat.memory.ChatMemory;
import org.springframework.ai.chat.memory.ChatMemoryRepository;
import org.springframework.ai.chat.memory.InMemoryChatMemoryRepository;
import org.springframework.ai.chat.memory.MessageWindowChatMemory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class MemoryConfig {
  @Bean
  public ChatMemoryRepository chatMemoryRepository() {
    return new InMemoryChatMemoryRepository();
  }

  @Bean
  public ChatMemory chatMemory(ChatMemoryRepository repo) {
    return MessageWindowChatMemory.builder()
        .chatMemoryRepository(repo)
        .maxMessages(100)
        .build();
  }
}
```

## Change the prompt template
`src/main/resources/prompt/build-analysis.st`
```
You are a CI/CD assistant that analyzes build, test, and deployment issues and can hold multi-turn conversations.

Respond ONLY in JSON:
response: short explanation
root_cause: detailed cause
recommendation: list of fixes
severity: [critical, warning, info]

Conversation history (if any):
{history}

Question:
{message}

Guidelines:
- Always return valid JSON (no Markdown).
- Keep recommendations clear and practical.
```


## Controller update
Add ChatMemory and it's import:
```java
import org.springframework.ai.chat.memory.ChatMemory;
```

```java
private final ChatMemory chatMemory;
public AIController(ChatClient chatClient, ChatMemory chatMemory) {
  this.chatClient = chatClient;
  this.chatMemory = chatMemory;
}
```

Add memory use and it's import:
```java
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.ai.chat.messages.UserMessage;
import org.springframework.ai.chat.messages.SystemMessage;
```

```java
@PostMapping("/{sessionId}")
public String ask(@PathVariable String sessionId, @RequestBody Map<String, String> request) {
  String message = request.get("message");

  // Retrieve chat history
  var history = chatMemory.get(sessionId);

  // Load the prompt template from the resource
  PromptTemplate template = new PromptTemplate(promptTemplate);
  var prompt = template.create(Map.of("history", history, "message", message));

  //Given chat memory here to Model 
  String response = chatClient.prompt(prompt)
      .advisors(advisorSpec -> advisorSpec.param("conversationId", sessionId)).call().content();

  // Update chat memory
  chatMemory.add(sessionId, new UserMessage(message));
  chatMemory.add(sessionId, new SystemMessage(response));
  return response;
}
```


## Test
```bash
curl -X POST http://localhost:8080/ask/123 -H "Content-Type: application/json" -d '{"message":"Hi, I’m debugging a Jenkins pipeline failure."}'
```
```bash
curl -X POST http://localhost:8080/ask/123 -H "Content-Type: application/json" -d '{"message":"What should I do next?"}'
```
```bash
curl -X POST http://localhost:8080/ask/123 -H "Content-Type: application/json" -d '{"message":"Can you summarize what we found so far?"}'
```


## Commit Changes to Git
After verifying the output, commit your changes:
```sh
git add src/main/java/com/example/spring_ai_demo/config/MemoryConfig.java src/main/java/com/example/spring_ai_demo/controller/AIController.java src/main/resources/prompt/build-analysis.st
git commit -m "add chat memory for contextual responses"
git push
```
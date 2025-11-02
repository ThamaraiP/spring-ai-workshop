# Step 1: Create REST Controller and Connect to LLM

## Objective
Create a Spring Boot REST controller that connects to the LLM using Spring AI.


## Code
```java

package com.example.spring_ai_demo.config;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.memory.ChatMemory;
import org.springframework.ai.chat.model.ChatModel;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ChatClientConfig {
  @Bean
  public ChatClient chatClient(ChatModel chatModel) {
    return ChatClient.builder(chatModel)
        .build();
  }
}

```


## Code
```java

package com.example.spring_ai_demo.controller;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.Map;

@RestController
@RequestMapping("/ask")
public class AIController {
  private final ChatClient chatClient;

  public AIController(ChatClient chatClient) {
    this.chatClient = chatClient;
  }

  @PostMapping
  public String ask(@RequestBody Map<String, String> request) {
    String message = request.get("message");
    return chatClient.prompt(message).call().content();
  }
}
```

## Test
```bash
curl -X POST http://localhost:8080/ask -H "Content-Type: application/json" -d '{"message":"Hi, I’m debugging a Jenkins pipeline failure."}'
```
```bash
curl -X POST http://localhost:8080/ask -H "Content-Type: application/json" -d '{"message":"What usually causes backend builds to fail"}'
```

## Commit Changes to Git
After verifying the output, commit your changes:
```sh
git add src/main/java/com/example/spring_ai_demo/config/ChatClientConfig.java src/main/java/com/example/springaidemo/controller/AIController.java
git commit -m "Add rest controller for prompting"
git push
```
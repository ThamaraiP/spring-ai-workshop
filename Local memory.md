# Step 3: Add Local Memory to Your AI Agent

This step continues from Step 2. Make sure you have implemented the ChatClient interface before proceeding.

## 1. Required Dependencies
Add the following dependencies to your `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-huggingface-spring-boot-starter</artifactId>
    <version>0.8.0</version>
</dependency>
```

## 2. application.properties
Ensure you have the following in `src/main/resources/application.properties`:
```
spring.ai.huggingface.api-token=YOUR_HUGGINGFACE_API_TOKEN
```
Replace `YOUR_HUGGINGFACE_API_TOKEN` with your actual token.

## 3. Why Do We Need Local Memory?
Local memory allows your AI agent to:
- Retain context across multiple interactions (multi-turn conversations)
- Personalize responses based on previous exchanges
- Improve user experience by remembering preferences or history

Without local memory, each prompt is treated in isolation, making it difficult to build intelligent, context-aware agents.

## 4. Implement Local Memory Service
Create a new Java class `LocalMemoryService.java` in your `src/main/java/com/example/springaidemo` package:

```java
import org.springframework.stereotype.Service;
import java.util.ArrayList;
import java.util.List;

@Service
public class LocalMemoryService {
    private final List<String> conversationHistory = new ArrayList<>();

    public void addMessage(String message) {
        conversationHistory.add(message);
    }

    public List<String> getHistory() {
        return new ArrayList<>(conversationHistory);
    }
}
```

## 5. Use Local Memory in ChatClientDemo
Update your `ChatClientDemo.java` to use the local memory service:

```java
import org.springframework.ai.chat.ChatClient;
import org.springframework.ai.chat.prompt.Prompt;
import org.springframework.boot.CommandLineRunner;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class ChatClientDemo implements CommandLineRunner {
    private final ChatClient chatClient;
    private final LocalMemoryService memoryService;

    @Autowired
    public ChatClientDemo(ChatClient chatClient, LocalMemoryService memoryService) {
        this.chatClient = chatClient;
        this.memoryService = memoryService;
    }

    @Override
    public void run(String... args) {
        // Simulate a multi-turn conversation
        Prompt prompt1 = new Prompt("Hello, AI! My name is Alex.");
        String response1 = chatClient.call(prompt1).getResult().getOutput();
        memoryService.addMessage("User: " + prompt1.getContents());
        memoryService.addMessage("AI: " + response1);
        System.out.println("First turn: " + response1);

        Prompt prompt2 = new Prompt("Can you remind me what my name is?");
        String response2 = chatClient.call(prompt2).getResult().getOutput();
        memoryService.addMessage("User: " + prompt2.getContents());
        memoryService.addMessage("AI: " + response2);
        System.out.println("Second turn: " + response2);

        // Show conversation history
        System.out.println("Conversation history:");
        memoryService.getHistory().forEach(System.out::println);
    }
}
```

## 6. Run the Application
Run your Spring Boot application:
```sh
./mvnw spring-boot:run
# or
./gradlew bootRun
```
You should see output showing:
- The AI's responses for each turn
- The full conversation history retained in local memory

## 7. Commit Changes to Git
After verifying the output, commit your changes:
```sh
git add src/main/java/com/example/springaidemo/LocalMemoryService.java src/main/java/com/example/springaidemo/ChatClientDemo.java
git commit -m "Add local memory service and demonstrate multi-turn conversation"
git push
```

Continue to Step 4: Add Persistent External Memory to your AI agent.

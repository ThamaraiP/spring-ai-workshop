# Step 2: Implement the ChatClient Interface

This step continues from Step 1. Make sure you have completed the blank Spring AI application setup before proceeding.

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
Create or update `src/main/resources/application.properties` and add:
```
spring.ai.huggingface.api-token=YOUR_HUGGINGFACE_API_TOKEN
```
Replace `YOUR_HUGGINGFACE_API_TOKEN` with your actual token.

## 3. Add ChatClient Implementation
Create a new Java class `ChatClientDemo.java` in your `src/main/java/com/example/springaidemo` package:

```java
import org.springframework.ai.chat.ChatClient;
import org.springframework.ai.chat.prompt.Prompt;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;
import java.util.List;

@Component
public class ChatClientDemo implements CommandLineRunner {
    private final ChatClient chatClient;

    public ChatClientDemo(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    @Override
    public void run(String... args) {
        // 1. call(Prompt prompt): Synchronous response
        Prompt prompt = new Prompt("Hello, AI! What can you do?");
        String response = chatClient.call(prompt).getResult().getOutput();
        System.out.println("call() response: " + response);

        // 2. stream(Prompt prompt): Streaming response
        System.out.println("stream() response:");
        chatClient.stream(prompt).forEach(token -> System.out.print(token.getOutput()));
        System.out.println();

        // 3. generate(List<Prompt> prompts): Multiple responses
        List<Prompt> prompts = List.of(new Prompt("Tell me a joke."), new Prompt("What's the weather today?"));
        chatClient.generate(prompts).forEach(result ->
            System.out.println("generate() response: " + result.getResult().getOutput())
        );
    }
}
```

## 4. Run the Application
Run your Spring Boot application from the command line:
```sh
./mvnw spring-boot:run
# or
./gradlew bootRun
```
You should see output in the terminal demonstrating the three ChatClient methods.

## 5. Commit Changes to Git
After verifying the output, commit your changes:
```sh
git add src/main/java/com/example/springaidemo/ChatClientDemo.java
git commit -m "Add ChatClient demo with call, stream, and generate methods"
git push
```

Continue to Step 3: Add Local Memory to your AI agent.

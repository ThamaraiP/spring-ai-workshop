# Step 4: Add Persistent External Memory to Your AI Agent

This step continues from Step 3. Make sure you have implemented local memory before proceeding.

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
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-chat-memory</artifactId>
    <version>0.8.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.7.1</version>
</dependency>
```

## 2. application.properties
Configure your database connection and Hugging Face token in `src/main/resources/application.properties`:
```
spring.datasource.url=jdbc:postgresql://localhost:5432/chatdb
spring.datasource.username=postgres
spring.datasource.password=postgres
spring.datasource.driver-class-name=org.postgresql.Driver

spring.ai.huggingface.api-token=YOUR_HUGGINGFACE_API_TOKEN
```
Replace `YOUR_HUGGINGFACE_API_TOKEN` with your actual token.

## 3. Docker Compose for PostgreSQL
Create a `docker-compose.yml` in your project root:
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: chatdb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```
Start the database:
```sh
docker compose up -d
```

## 4. Code Snippet: Persistent Chat Memory Integration
Create or update your chat agent configuration (e.g., `ChatMemoryConfig.java`):

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.ai.chat.memory.JdbcChatMemoryRepository;
import org.springframework.ai.chat.client.advisor.MessageChatMemoryAdvisor;

@Configuration
public class ChatMemoryConfig {
    @Bean
    public JdbcChatMemoryRepository jdbcChatMemoryRepository(JdbcTemplate jdbcTemplate) {
        return new JdbcChatMemoryRepository(jdbcTemplate);
    }
    @Bean
    public MessageChatMemoryAdvisor messageChatMemoryAdvisor(JdbcChatMemoryRepository repository) {
        return new MessageChatMemoryAdvisor(repository);
    }
}
```

Update your chat client runner to use the advisor (e.g., `ChatClientDemo.java`):

```java
import org.springframework.ai.chat.ChatClient;
import org.springframework.ai.chat.prompt.Prompt;
import org.springframework.ai.chat.client.advisor.MessageChatMemoryAdvisor;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class ChatClientDemo implements CommandLineRunner {
    private final ChatClient chatClient;
    private final MessageChatMemoryAdvisor memoryAdvisor;

    public ChatClientDemo(ChatClient chatClient, MessageChatMemoryAdvisor memoryAdvisor) {
        this.chatClient = chatClient;
        this.memoryAdvisor = memoryAdvisor;
    }

    @Override
    public void run(String... args) {
        Prompt prompt = new Prompt("Hello, AI! Remember this message.");
        String response = chatClient.call(prompt, memoryAdvisor).getResult().getOutput();
        System.out.println("Response: " + response);

        Prompt followUp = new Prompt("What did I just say?");
        String followUpResponse = chatClient.call(followUp, memoryAdvisor).getResult().getOutput();
        System.out.println("Follow-up Response: " + followUpResponse);
    }
}
```

## 5. Run the Application
1. Ensure PostgreSQL is running (`docker compose up -d`).
2. Start your Spring Boot app:
   ```sh
   ./mvnw spring-boot:run
   # or
   ./gradlew bootRun
   ```
3. Verify output: The agent should remember previous messages across runs.

## 6. Commit Changes to Git
After verifying the output, commit your changes:
```sh
git add src/main/java/com/example/springaidemo/ChatMemoryConfig.java src/main/java/com/example/springaidemo/ChatClientDemo.java src/main/resources/application.properties docker-compose.yml
git commit -m "Add persistent chat memory with JdbcChatMemoryRepository and PostgreSQL"
git push
```

You now have persistent external memory for your chat agent using Spring AI and PostgreSQL. This enables your agent to remember conversations across sessions and restarts.

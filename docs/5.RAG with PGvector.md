# Step 5: Add RAG with PGvector to Your AI Agent

This step continues from Step 4 (External Memory). Make sure you have completed the previous steps before proceeding.

## 1. Introduction to RAG and PGvector
Retrieval-Augmented Generation (RAG) enhances large language models (LLMs) by integrating them with external data sources, enabling more accurate and contextually relevant responses. RAG uses embedding models to convert text into vector representations, which are stored in a vector database. PGvector is an open-source PostgreSQL extension for storing and searching embeddings.

## 2. Get Your Hugging Face API Token
To use Spring AI with Hugging Face, you need an API token:
- Go to [Hugging Face Settings - Access Tokens](https://huggingface.co/settings/tokens)
- Click "New token" and create a token with the required scopes
- Copy your token for use in the next step

---

## 3. Required Dependencies
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
    <artifactId>spring-ai-pgvector-store</artifactId>
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

---

## 4. application.properties
Configure your database connection and Hugging Face token in `src/main/resources/application.properties`:
```
spring.datasource.url=jdbc:postgresql://localhost:5432/chatdb
spring.datasource.username=postgres
spring.datasource.password=postgres
spring.datasource.driver-class-name=org.postgresql.Driver

spring.ai.huggingface.api-token=YOUR_HUGGINGFACE_API_TOKEN
```
Replace `YOUR_HUGGINGFACE_API_TOKEN` with the token you obtained above.

---

## 5. Docker Compose for PostgreSQL with PGvector
Create a `docker-compose.yml` in your project root:
```yaml
version: '3.8'
services:
  postgres:
    image: ankane/pgvector:latest
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

---

## 6. Load Documents Upfront into the Vector Store
Before you can use RAG to answer questions, you must load your documents into the vector store. This step ensures the agent has context to retrieve relevant information.

Add a startup runner to load documents:

```java
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.ai.document.Document;
import org.springframework.boot.ApplicationRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.List;

@Configuration
public class DocumentLoaderConfig {
    @Bean
    public ApplicationRunner loadDocuments(VectorStore vectorStore) {
        return args -> {
            List<Document> docs = List.of(
                new Document("doc1", "RAG stands for Retrieval-Augmented Generation. It combines LLMs with external knowledge bases for better answers."),
                new Document("doc2", "PGvector is a PostgreSQL extension for storing and searching vector embeddings, useful for semantic search in RAG.")
            );
            vectorStore.add(docs);
        };
    }
}
```

---

## 7. Code Snippet: RAG Integration with VectorStore
Create or update your RAG configuration (e.g., `RagConfig.java`):

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.ai.chat.client.advisor.vectorstore.QuestionAnswerAdvisor;

@Configuration
public class RagConfig {
    @Bean
    public QuestionAnswerAdvisor questionAnswerAdvisor(VectorStore vectorStore) {
        return new QuestionAnswerAdvisor(vectorStore);
    }
}
```

Update your chat client runner to use the advisor (e.g., `ChatClientDemo.java`):

```java
import org.springframework.ai.chat.ChatClient;
import org.springframework.ai.chat.prompt.Prompt;
import org.springframework.ai.chat.client.advisor.vectorstore.QuestionAnswerAdvisor;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class ChatClientDemo implements CommandLineRunner {
    private final ChatClient chatClient;
    private final QuestionAnswerAdvisor ragAdvisor;

    public ChatClientDemo(ChatClient chatClient, QuestionAnswerAdvisor ragAdvisor) {
        this.chatClient = chatClient;
        this.ragAdvisor = ragAdvisor;
    }

    @Override
    public void run(String... args) {
        Prompt prompt = new Prompt("What is RAG and how does PGvector help?");
        String response = chatClient.call(prompt, ragAdvisor).getResult().getOutput();
        System.out.println("RAG Response: " + response);
    }
}
```

---

## 8. Run the Application
1. Ensure PostgreSQL with PGvector is running (`docker compose up -d`).
2. Start your Spring Boot app:
   ```sh
   ./mvnw spring-boot:run
   # or
   ./gradlew bootRun
   ```
3. Verify output: The agent should use RAG to answer queries using the vector store.

---

## 9. Commit Changes to Git
After verifying the output, commit your changes:
```sh
git add src/main/java/com/example/springaidemo/DocumentLoaderConfig.java src/main/java/com/example/springaidemo/RagConfig.java src/main/java/com/example/springaidemo/ChatClientDemo.java src/main/resources/application.properties docker-compose.yml
git commit -m "Add RAG with PGvector and upfront document loading to AI agent"
git push
```

---

You have now enhanced your AI agent with RAG capabilities using PGvector, enabling retrieval-augmented generation from your vector database with documents loaded upfront.

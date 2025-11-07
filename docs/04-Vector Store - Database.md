# Vector Store with PGVector

## Objective

In this section, we will enhance our Spring AI application by integrating a vector store to enable
log-based context search using embeddings. This will allow our AI assistant to provide more accurate
and context-aware responses based on historical log data.

## prequisites

- Complete the previous sections of the workshop.
- PostgreSQL database with `pgvector` extension installed.

| Platform / Method | Installation / Vector Support                        | Create Database                                                                                                                                                    | Notes / Connection Info                                                                                            |
|-------------------|------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
| **🐳 Docker**     | Use `ankane/pgvector:latest`                         | Auto-created if `POSTGRES_DB=chat_memory_db` or:<br>`docker exec -it postgres_vector_db psql -U postgres -c "CREATE DATABASE chat_memory_db OWNER postgres;"`      | Vector support built-in.<br>Connect via:<br>`postgresql://postgres:mysecretpassword@localhost:5432/chat_memory_db` |
| **🍎 macOS**      | `brew install postgresql`<br>`brew install pgvector` | `sql CREATE ROLE postgres WITH LOGIN SUPERUSER PASSWORD 'mysecretpassword'; CREATE DATABASE chat_memory_db OWNER postgres; CREATE EXTENSION IF NOT EXISTS vector;` | Adds `vector` types to your database                                                                               |
| **🪟 Windows**    | Official installer + manual `pgvector` compilation   | Same database creation, then:<br>`CREATE EXTENSION vector;` (after installing vector)                                                                              | Easier to use Docker on Windows for vector support                                                                 |

## Docker Compose Example

Create a `docker-compose.yml` file with the following content:

```yaml
version: '3.8'

services:
  postgres:
    image: ankane/pgvector:latest   # This image includes the pgvector extension
    container_name: postgres_vector_db
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: mysecretpassword
      POSTGRES_DB: chat_memory_db    # This database will be created automatically
    ports:
      - "5432:5432"
    volumes:
      - ./src/main/resources/db/init.sql:/docker-entrypoint-initdb.d/init.sql
```

Database initialization script (`init.sql`):

```sql
CREATE
EXTENSION IF NOT EXISTS vector;
```

## pom.xml

```xml

<dependency>
  <groupId>org.springframework.ai</groupId>
  <artifactId>spring-ai-starter-vector-store-pgvector</artifactId>
  <version>1.0.3</version>
</dependency>
```

## application.properties

```properties
# PostgreSQL configuration
spring.datasource.url=jdbc:postgresql://localhost:5432/chat_memory_db
spring.datasource.username=postgres
spring.datasource.password=mysecretpassword
# Hibernate auto-create schema
spring.jpa.hibernate.ddl-auto=update
# Explicitly tell Hibernate which SQL dialect to use
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect

```

## Vector Configuration

```java
package com.example.spring_ai_demo.config;

import org.springframework.ai.embedding.EmbeddingModel;
import org.springframework.ai.vectorstore.pgvector.PgVectorStore;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;

@Configuration
public class VectorConfig {

  @Bean
  public PgVectorStore pgVectorStore(JdbcTemplate jdbcTemplate, EmbeddingModel embeddingModel) {
    return PgVectorStore.builder(jdbcTemplate, embeddingModel)
        .vectorTableName("documents")
        .dimensions(1536)
        .build();
  }
}
```

## Load logs

Copy the dev-docs folder into your src/main/resources directory.
See [dev-docs](../dev-docs) for the actual folder location.

```java
package com.example.spring_ai_demo.service;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.ai.document.Document;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.boot.CommandLineRunner;
import org.springframework.core.io.ClassPathResource;
import org.springframework.stereotype.Component;

@Component
public class LogLoader implements CommandLineRunner {

  private static final Logger log = LoggerFactory.getLogger(LogLoader.class);
  private final VectorStore vectorStore;

  private static final String DOCS_FOLDER = "dev-docs";

  public LogLoader(VectorStore vectorStore) {
    this.vectorStore = vectorStore;
  }

  @Override
  public void run(String... args) {
    List<Document> docs = loadDocuments();
    if (!docs.isEmpty()) {
      vectorStore.add(docs);
      System.out.println("✅ Loaded " + docs.size() + " documents into Vector Store.");
    } else {
      System.err.println("⚠️ No documents found in " + DOCS_FOLDER);
    }
    log.info("✅ Vector store populated successfully.");
  }


  private List<Document> loadDocuments() {
    List<Document> documents = new ArrayList<>();
    ClassPathResource folderResource = new ClassPathResource(DOCS_FOLDER);
    Path folderPath = null;
    try {
      folderPath = folderResource.getFile().toPath();
    } catch (IOException e) {
      log.error("Failed to load folder {}", DOCS_FOLDER, e);
    }

    try {
      assert folderPath != null;
      Files.walk(folderPath)
          .filter(Files::isRegularFile)
          .forEach(path -> {
            try {
              String content = Files.readString(path);
              documents.add(new Document(content,
                  Map.of("filename", path.getFileName().toString()
                  )));
            } catch (IOException e) {
              log.error("Failed to load file {}", path.getFileName().toString(), e);
            }
          });
    } catch (IOException e) {
      log.error("Failed to load folder {}", DOCS_FOLDER, e);
    }
    return documents;
  }
}

```

## Update Controller to include context

Add VectorStore and it's import:

```java
import org.springframework.ai.vectorstore.VectorStore;
```

```java
private final VectorStore vectorStore;

public AIController(ChatClient chatClient, ChatMemory chatMemory, VectorStore vectorStore) {
  this.chatClient = chatClient;
  this.chatMemory = chatMemory;
  this.vectorStore = vectorStore;
}
```

Add import related to VectorStore:

```java
import org.springframework.ai.document.Document;
import org.springframework.ai.vectorstore.SearchRequest;
import java.util.List;
import java.util.stream.Collectors;
```

```java
public String ask(@PathVariable String sessionId, @RequestBody Map<String, String> request) {
  String message = request.get("message");

  // Retrieve chat history
  var history = chatMemory.get(sessionId);

  // Perform similarity search in vector store
  List<Document> docs = vectorStore.similaritySearch(SearchRequest.builder()
      .query(message).topK(5).build());
  String context = docs.stream().map(Document::getText).collect(Collectors.joining("\n---\n"));

  // Load the prompt template from the resource
  PromptTemplate template = new PromptTemplate(promptTemplate);
  var prompt = template.create(Map.of("history", history, "context", context, "message", message));

  //Given tools definition here to Model along with Memory and Context
  String response = chatClient.prompt(prompt)
      .advisors(advisorSpec -> advisorSpec.param("conversationId", sessionId)).call().content();

  // Update chat memory
  chatMemory.add(sessionId, new UserMessage(message));
  assert response != null;
  chatMemory.add(sessionId, new SystemMessage(response));
  return response;
}
  ```

## Change the prompt template

`src/main/resources/prompt/build-analysis.st`

```
You are a CI/CD assistant that analyzes build, test, and deployment issues using the provided context from log files along with chat history.

Respond ONLY in JSON:
response: short explanation
root_cause: detailed cause
recommendation: list of fixes
severity: [critical, warning, info]
related_files: list of affected files

Conversation history (if any):
{history}

Relevant context from logs or documents (if any, with filenames):
{context}

Note: When forming "related_files", prefer filenames mentioned above (e.g., from File: ... sections).

Question:
{message}

Guidelines:
- Always return valid JSON (no Markdown).
- Keep recommendations clear and practical.
```

## Test

```bash
curl -X POST http://localhost:8080/ask/123 -H "Content-Type: application/json" -d '{"message":"Why did my backend build fail"}'
```

```bash
curl -X POST http://localhost:8080/ask/123 -H "Content-Type: application/json" -d '{"message":"Which test exactly failed"}'
```

```bash
curl -X POST http://localhost:8080/ask/123 -H "Content-Type: application/json" -d '{"message":"Can you summarize all failures in the last Jenkins pipeline?"}'
```

```bash
curl -X POST http://localhost:8080/ask/123 -H "Content-Type: application/json" -d '{"message":"How can I fix the NullPointerException?"}'
```

```bash
curl -X POST http://localhost:8080/ask/123 -H "Content-Type: application/json" -d '{"message":"Was the deployment successful?"}'
```

## Commit Changes to Git

After verifying the output, commit your changes:

```sh
git add src/main/java/com/example/spring_ai_demo/config/VectorConfig.java src/main/java/com/example/spring_ai_demo/service/LogLoader.java src/main/java/com/example/spring_ai_demo/controller/AIController.java src/main/resources/prompt/build-analysis.st
git commit -m "add vector store for log-based context search"
git push
```

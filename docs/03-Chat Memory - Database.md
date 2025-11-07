# Introduce Chat Memory With Postgres

## Objective

Extend the chat memory to use a **persistent PostgreSQL database** instead of in-memory storage, so
that conversation history survives restarts and scales across instances.

## Prerequisites

- A running PostgreSQL instance (locally via Docker or a managed service)
- Database and user created for the application

## PostgreSQL Installation Options

| Platform / Method          | Installation Command                                                                                  | Start Command                              | Create Database Command                                                                                                                                                                                    | Notes / Connection Info                                                                                                                       |
  |----------------------------|-------------------------------------------------------------------------------------------------------|--------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| **🐳 Docker**              | *(No manual install needed)*<br>Use `docker-compose.yml`<br>(see below)                               | `docker compose up -d`                     | Auto-created if `POSTGRES_DB=chat_memory_db` is set in `docker-compose.yml`<br><br>Otherwise:<br>`docker exec -it postgres_vector_db psql -U postgres -c "CREATE DATABASE chat_memory_db OWNER postgres;"` | Image: `ankane/pgvector:latest` (includes pgvector)<br>Connect via:<br>`postgresql://postgres:mysecretpassword@localhost:5432/chat_memory_db` |
| **🍎 macOS (Homebrew)**    | `brew update && brew install postgresql`                                                              | `brew services start postgresql`           | `bash psql postgres -c "CREATE ROLE postgres WITH LOGIN SUPERUSER PASSWORD 'mysecretpassword';" psql postgres -c "CREATE DATABASE chat_memory_db OWNER postgres;"`                                         | Confirm service with:<br>`brew services list`<br>Connect via:<br>`postgresql://postgres:mysecretpassword@localhost:5432/chat_memory_db`       |
| **🪟 Windows (Installer)** | Download installer:<br>[postgresql.org/download/windows](https://www.postgresql.org/download/windows) | Service starts automatically after install | Open “SQL Shell (psql)” and run:<br>`CREATE DATABASE chat_memory_db OWNER postgres;`                                                                                                                       | Default user `postgres` created during install.<br>Connect via:<br>`postgresql://postgres:<your_password>@localhost:5432/chat_memory_db`      |

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
CREATE EXTENSION IF NOT EXISTS vector;
```

## pom.xml additions

```xml

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
<groupId>org.postgresql</groupId>
<artifactId>postgresql</artifactId>
<scope>runtime</scope>
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

## Entity class

```java
package com.example.spring_ai_demo.gateway.postgres.chatMemory.entity;


import jakarta.persistence.*;

@Entity
public class ChatMessageEntity {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  public ChatMessageEntity(String conversationId, String role, String content) {
    this.conversationId = conversationId;
    this.role = role;
    this.content = content;
  }

  public ChatMessageEntity() {
  }

  public Long getId() {
    return id;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public String getConversationId() {
    return conversationId;
  }

  public void setConversationId(String conversationId) {
    this.conversationId = conversationId;
  }

  public String getRole() {
    return role;
  }

  public void setRole(String role) {
    this.role = role;
  }

  public String getContent() {
    return content;
  }

  public void setContent(String content) {
    this.content = content;
  }

  private String conversationId;
  private String role;

  @Column(columnDefinition = "TEXT")
  private String content;
}

```
## Repository

```java
package com.example.spring_ai_demo.gateway.postgres.chatMemory;

import com.example.spring_ai_demo.gateway.postgres.chatMemory.entity.ChatMessageEntity;
import java.util.List;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ChatMessageRepository extends JpaRepository<ChatMessageEntity, Long> {
  List<ChatMessageEntity> findByConversationId(String conversationId);
  void deleteByConversationId(String conversationId);
}

```

## Repository Implementation

```java
package com.example.spring_ai_demo.gateway.postgres.chatMemory.repository;

import com.example.spring_ai_demo.gateway.postgres.chatMemory.ChatMessageRepository;
import com.example.spring_ai_demo.gateway.postgres.chatMemory.entity.ChatMessageEntity;
import java.util.Comparator;
import java.util.List;
import java.util.stream.Collectors;
import org.springframework.ai.chat.memory.ChatMemoryRepository;
import org.springframework.ai.chat.messages.AssistantMessage;
import org.springframework.ai.chat.messages.Message;
import org.springframework.ai.chat.messages.SystemMessage;
import org.springframework.ai.chat.messages.UserMessage;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

@Repository
public class PostgresMemoryRepositoryImpl implements ChatMemoryRepository {

  private final ChatMessageRepository repository;

  public PostgresMemoryRepositoryImpl(ChatMessageRepository repository) {
    this.repository = repository;
  }

  @Override
  @Transactional(readOnly = true)
  public List<String> findConversationIds() {
    return repository.findAll().stream()
        .map(ChatMessageEntity::getConversationId)
        .distinct()
        .toList();
  }

  @Override
  @Transactional(readOnly = true)
  public List<Message> findByConversationId(String conversationId) {
    return repository.findByConversationId(conversationId).stream()
        .sorted(Comparator.comparing(ChatMessageEntity::getId))
        .map(e -> switch (e.getRole().toLowerCase()) {
          case "user" -> new UserMessage(e.getContent());
          case "assistant" -> new AssistantMessage(e.getContent());
          default -> new SystemMessage(e.getContent());
        })
        .collect(Collectors.toList());
  }

  @Override
  @Transactional
  public void saveAll(String conversationId, List<Message> messages) {
    // Remove old records before saving
    repository.deleteByConversationId(conversationId);

    List<ChatMessageEntity> entities = messages.stream()
        .map(m -> buildEntity(conversationId, m))
        .toList();

    repository.saveAll(entities);
  }

  private static ChatMessageEntity buildEntity(String conversationId, Message m) {
    return new ChatMessageEntity(conversationId, m.getMessageType().name().toLowerCase(), m.getText());
  }

  @Override
  @Transactional
  public void deleteByConversationId(String conversationId) {
    repository.deleteByConversationId(conversationId);
  }
}


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
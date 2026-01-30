# Introduce Tool Calling

## Objective
Enable LLM-triggered actions like auto-fix or rerun Jenkins.

![Tool Calling.png](../images/Tool%20Calling.png)

## Tool implementation
```java
package com.example.spring_ai_demo.service;

import org.springframework.ai.tool.annotation.Tool;
import org.springframework.stereotype.Component;

@Component
public class CICDTools {

  @Tool(name = "commitFix", description = "Commit fix to Git repository")
  public String commitFix(String fileName, String fixDescription) {
    return "✅ Mock: Committed fix to " + fileName + " with message: " + fixDescription;
  }

  @Tool(name = "rerunPipeline", description ="Rerun Jenkins pipeline")
  public String rerunPipeline() {
    return "🚀 Mock: Jenkins pipeline rerun triggered successfully.";
  }
}


```

## Update Controller
Add CICDTools and it's import:
```java
import com.example.spring_ai_demo.service.CICDTools;
```

```java
private final CICDTools cicdTools;
public AIController(ChatClient chatClient, ChatMemory chatMemory, VectorStore vectorStore, CICDTools cicdTools) {
  this.chatClient = chatClient;
  this.chatMemory = chatMemory;
  this.vectorStore = vectorStore;
  this.cicdTools = cicdTools;
}
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
      .tools(cicdTools)
      .advisors(advisorSpec -> advisorSpec.param("conversationId", sessionId)).call().content();

  // Update chat memory
  chatMemory.add(sessionId, new UserMessage(message));
  assert response != null;
  chatMemory.add(sessionId, new AssistantMessage(response));
  return response;
}
```

## Change the prompt template 
`src/main/resources/prompt/build-analysis.st`
```
You are a CI/CD assistant that analyzes build, test, and deployment issues using the provided context from log files along with chat history and tools

Respond ONLY in JSON:
response: short explanation
root_cause: detailed cause
recommendation: list of fixes
severity: [critical, warning, info]
related_files: list of affected files
tools_triggered: list of tool names invoked if any

Conversation history (if any):
{history}

Relevant context from logs or documents (if any, with filenames):
{context}

Note: When forming "related_files", prefer filenames mentioned above (e.g., from File: ... sections).

Question:
{message}

Guidelines:
- Only call tools when user explicitly requests an action (like "fix", "commit", or "rerun").
- Always return valid JSON (no Markdown).
- Keep recommendations clear and practical.
```
## Change to Response Entity

Creating the record AnalysisResponse
```java
package com.example.spring_ai_demo.model;

import java.util.List;

public record AnalysisResponse(String response, String root_cause, List<String> recommendation,
                               String severity) {

}
```

```java
public AnalysisResponse ask(@PathVariable String sessionId, @RequestBody Map<String, String> request) {
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
      .tools(cicdTools)
      .advisors(advisorSpec -> advisorSpec.param("conversationId", sessionId)).call().entity(AnalysisResponse.class);

  // Update chat memory
  chatMemory.add(sessionId, new UserMessage(message));
  assert response != null;
  chatMemory.add(sessionId, new AssistantMessage(response));
  return response;
}
```

## Example query
```bash
curl -X POST http://localhost:8080/ask/123 -H "Content-Type: application/json" -d '{"message":"Was the deployment successful?"}'
```

```bash
curl -X POST http://localhost:8080/ask/123 -H "Content-Type: application/json" -d '{"message":"commit the fix and run the jerkins pipeline?"}'
```


## Commit Changes to Git
After verifying the output, commit your changes:
```sh
git add src/main/java/com/example/spring_ai_demo/service/CICDTools.java src/main/java/com/example/spring_ai_demo/controller/AIController.java src/main/resources/prompt/build-analysis.st
git commit -m "add tool calling for auto-fix and pipeline rerun"
git push
```

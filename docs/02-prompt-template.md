# Step 2: Introduce Prompt Template

## Objective
Use a `.st` file for structured prompting.

## Add prompt template resource
`src/main/resources/prompt/build-analysis.st`
```
Respond ONLY in JSON:
response: short explanation
root_cause: detailed cause
recommendation: list of fixes
severity: [critical, warning, info]

Question:
{message}
```


## Add PromptTemplate to Controller
```java
@Value("classpath:prompt/build-analysis.st")
private Resource promptTemplate;
```

## Update the endpoint to use PromptTemplate
```java
@PostMapping
public String ask(@RequestBody Map<String, String> req) {
  String message = req.get("message");
  
  PromptTemplate template = new PromptTemplate(promptTemplate);
  var prompt = template.create(Map.of("message", message));
  
  return chatClient.prompt(prompt).call().content();
}
```

## Add Import
```java
import org.springframework.core.io.Resource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.ai.prompt.template.PromptTemplate;
```

## Test
```bash
curl -X POST http://localhost:8080/ask -H "Content-Type: application/json" -d '{"message":"Hi, I’m debugging a Jenkins pipeline failure."}'
```
```bash
curl -X POST http://localhost:8080/ask -H "Content-Type: application/json" -d '{"message":"What should I do next?"}'
```
```bash
curl -X POST http://localhost:8080/ask -H "Content-Type: application/json" -d '{"message":"Can you summarize what we found so far?"}'
```

## Commit Changes to Git
After verifying the output, commit your changes:
```sh
git add src/main/java/com/example/spring_ai_demo/controller/AIController.java src/main/resources/prompt/build-analysis.st
git commit -m "Add prompt template resource and update controller to use it"
git push
```

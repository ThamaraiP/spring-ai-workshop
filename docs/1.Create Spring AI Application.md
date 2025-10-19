# Step 1: Create a Blank Spring AI Application

This is the first step in building your Spring AI agent from scratch. Follow these instructions before moving to the next step.

## 1. Get Your Hugging Face API Token
To use Spring AI with Hugging Face, you need an API token:
- Go to [Hugging Face Settings - Access Tokens](https://huggingface.co/settings/tokens)
- Click "New token" and create a token with the required scopes
- Copy your token for use in the next step

---

## 2. application.properties
Create `src/main/resources/application.properties` and add:
```
spring.ai.huggingface.api-token=YOUR_HUGGINGFACE_API_TOKEN
```
Replace `YOUR_HUGGINGFACE_API_TOKEN` with the token you obtained above.

---

## 3. Project Setup
Use the [Spring Initializr](https://start.spring.io/) UI to generate your project:

- **Project:** Maven or Gradle
- **Language:** Java
- **Spring Boot Version:** (latest stable)
- **Group:** com.example (or your choice)
- **Artifact:** spring-ai-demo (or your choice)
- **Name:** spring-ai-demo
- **Description:** Blank Spring AI application
- **Package name:** com.example.springaidemo (or your choice)
- **Packaging:** Jar
- **Java Version:** 17 or later

### Add Dependencies
- **Spring Web**
- **Spring Boot Actuator** (optional)
- **Spring AI Hugging Face**

Click "Generate" to download the project zip. Unzip and open it in your IDE (IntelliJ IDEA recommended).

---

## 4. Run the Blank Application
In your IDE or terminal, run the application:

For Maven:
```sh
./mvnw spring-boot:run
```
For Gradle:
```sh
./gradlew bootRun
```

The application should start with no errors. You can check the default actuator endpoint at `http://localhost:8080/actuator` (if enabled).

---

## 5. Commit to Your Git Account
1. Initialize a git repository (if not already done):
   ```sh
git init
git add .
git commit -m "Initial blank Spring AI application"
```
2. Create a new repository on your Git hosting service (e.g., GitHub, GitLab).
3. Add the remote and push your code:
   ```sh
git remote add origin <your-repo-url>
git push -u origin main
```

---

Your blank Spring AI application is now ready and committed to your git account. Continue to Step 2: Implement the ChatClient interface.

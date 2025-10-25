# Pre-requisites for Spring AI Workshop

To ensure a smooth and productive experience during the Spring AI Workshop, participants are required to complete the following setup prior to the session:

## 1. OpenAI API Key
- You will need an active OpenAI API key to interact with OpenAI services.
- Sign up or log in at [OpenAI](https://platform.openai.com/) to generate your API key.
- Keep your API key handy as it will be needed during hands-on exercises.

## 2. PostgreSQL Database Setup
- A PostgreSQL database is required for storing and querying structured data.
- You can install PostgreSQL via Docker or any equivalent local setup.

### Using Docker:
```bash
docker run --name spring-ai-postgres -e POSTGRES_PASSWORD=yourpassword -p 5432:5432 -d postgres
```
Replace `yourpassword` with a secure password of your choice.

- Alternatively, you can install PostgreSQL natively on your system if preferred.
- Confirm that PostgreSQL is running and accessible on your local machine.

## 3. GitHub Access Key / Personal Access Token
- A GitHub access token is required to interact with repositories programmatically.
- Generate a token with `repo` and `workflow` permissions:
  1. Go to [GitHub Settings → Developer settings → Personal Access Tokens → Tokens (classic)](https://github.com/settings/tokens).
  2. Click **Generate new token**, select necessary scopes, and copy the token.
- Store this token securely; it will be used to clone or push to repositories during exercises.

## 4. Additional Requirements
- Ensure your system has **Docker installed** if you plan to use Docker for PostgreSQL.
- Confirm your machine has **Java 17+** installed for Spring Boot compatibility.
- Recommended: An IDE like **IntelliJ IDEA** or **VS Code** with Java and Spring Boot support.

## Checklist Before the Workshop
- [ ] OpenAI API Key ready
- [ ] PostgreSQL running (via Docker or local installation)
- [ ] GitHub access token generated and ready
- [ ] Java 17+ installed and configured
- [ ] IDE ready for Spring Boot development

Completing these steps beforehand will allow you to fully participate in all workshop exercises without delays.


# spring-ai-workshop

This repository presents a comprehensive, modular workshop for building production-grade AI agents using the Spring AI framework. The approach is designed to guide practitioners, architects, and engineers through the full lifecycle of AI agent development, from initial project scaffolding to advanced retrieval-augmented generation (RAG) with persistent vector stores.

## Approach and Philosophy

- **Incremental, Hands-On Learning:**
  Each step in the `docs` folder is crafted as a self-contained module, allowing you to build, test, and understand one capability at a time. This enables both linear progression for newcomers and selective deep-dives for experienced developers.

- **Best Practices by Default:**
  The workshop enforces modern Spring Boot and Spring AI conventions, including dependency management, configuration via `application.properties`, and idiomatic use of dependency injection, service layers, and advisor patterns.

- **Real-World AI Agent Patterns:**
  The progression covers:
  - Project bootstrapping and environment setup
  - Integration of LLMs via Spring AI
  - Synchronous and streaming chat interfaces
  - Local and persistent memory (JDBC, PGvector)
  - Retrieval-augmented generation (RAG) with vector search
  - Secure API key management and external service integration

- **Reproducibility and Extensibility:**
  All steps are versioned and documented, with explicit instructions for running, testing, and committing code. The modular structure allows you to extend or adapt each capability for your own use cases, including plugging in different LLM providers, vector stores, or memory strategies.

- **Production-Ready Foundations:**
  The workshop emphasizes scalable, maintainable code, including Docker-based infrastructure for PostgreSQL/PGvector, robust configuration, and clear separation of concerns. Each code snippet is ready for integration into real-world microservices or enterprise applications.

## Structure

- **docs/**: Contains sequential, numbered guides for each capability:
  - Project setup and API key management
  - ChatClient usage and prompt engineering
  - Local memory and multi-turn conversation
  - Persistent memory with JDBC/PGvector
  - RAG with document loading and semantic search
- **src/**: Example Java code for each step, following best practices
- **docker-compose.yml**: Infrastructure for running required services locally

## Getting Started
- [Step 1: Create Spring AI Application](docs/1.Create%20Spring%20AI%20Application.md)

Follow the sequence in the `docs` folder to incrementally develop, test, and extend your Spring AI agent.

For detailed framework reference, see the [Spring AI documentation](https://docs.spring.io/spring-ai/docs/current/reference/html/index.html).

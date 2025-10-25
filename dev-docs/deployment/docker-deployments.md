# Docker Deployment Guidelines

- Use multi-stage builds to reduce image size.
- Keep `ENTRYPOINT` as the main application start command.
- Store configuration externally using environment variables.
- Verify port mappings align with Spring Boot server port (`server.port`).

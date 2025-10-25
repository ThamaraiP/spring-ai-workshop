# Common Maven Build Failures

1. **Dependency Resolution Errors:** Check `settings.xml` for correct repository URLs.
2. **Test Failures:** Run `mvn test -X` to identify root cause.
3. **Plugin Version Issues:** Ensure plugin versions align with your Spring Boot parent POM.
4. **OutOfMemoryError:** Increase Maven memory via `MAVEN_OPTS=-Xmx1G`.

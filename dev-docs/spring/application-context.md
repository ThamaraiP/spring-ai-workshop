# Understanding ApplicationContext

The ApplicationContext is the central interface for accessing Spring beans.
Common causes of startup failure:
- Misconfigured component scanning paths.
- Circular dependencies between beans.
- Missing configuration files under `resources/`.

You can debug using `--debug` flag or enabling `spring.main.log-startup-info=true`.

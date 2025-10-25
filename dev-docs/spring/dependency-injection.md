# Spring Dependency Injection

Spring manages dependencies through the ApplicationContext, which instantiates, configures, and assembles beans.

If you encounter `No qualifying bean of type`:
- Ensure the bean is annotated with `@Component`, `@Service`, or `@Bean`.
- Verify that component scanning includes the package.
- Check that configuration classes are loaded properly.

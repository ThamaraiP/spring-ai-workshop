# Bean Lifecycle in Spring

1. Instantiation of bean.
2. Property population.
3. `@PostConstruct` or `InitializingBean.afterPropertiesSet()`.
4. Bean is ready for use.
5. `@PreDestroy` or `DisposableBean.destroy()` on shutdown.

If resources are not released properly, verify `@PreDestroy` annotations are used.

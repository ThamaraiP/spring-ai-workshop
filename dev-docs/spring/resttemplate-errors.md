# Common RestTemplate Errors

If you see `No qualifying bean of type 'RestTemplateBuilder'`, you must define a RestTemplate bean.

Example fix:
```java
@Bean
public RestTemplate restTemplate(RestTemplateBuilder builder) {
    return builder.build();
}
```

Alternatively, migrate to WebClient from Spring WebFlux for reactive, non-blocking HTTP calls.

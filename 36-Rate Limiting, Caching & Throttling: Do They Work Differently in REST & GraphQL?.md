# Rate Limiting, Caching & Throttling: Do They Work Differently in REST & GraphQL?

In this article, we’ll explore how **rate limiting**, **caching**, and **throttling** work in both **RESTful APIs** and **GraphQL APIs** within the context of **Spring Boot**, a popular Java framework. We’ll break down how these concepts differ between the two paradigms and provide practical examples using Spring Boot’s features.

---

## 1. Rate Limiting: REST vs GraphQL in Spring Boot

### What is Rate Limiting?

Rate limiting restricts the number of requests a client can make within a specific time window to prevent abuse and ensure fair usage.

---

### **REST Example: Spring Boot + Spring Cloud Gateway**

In a traditional REST API, rate limiting is often implemented using **Spring Cloud Gateway** or a custom filter. For simplicity, we’ll use the `spring-rate-limit` library.

#### Step 1: Add Dependencies

Add the following to your `pom.xml`:

```xml
<dependency>
    <groupId>com.github.micvang</groupId>
    <artifactId>spring-rate-limit</artifactId>
    <version>1.0.4</version>
</dependency>
```

#### Step 2: Configure Rate Limiting

```java
@Configuration
public class RateLimitConfig {

    @Bean
    public RateLimiter rateLimiter() {
        return new RateLimiter("myApp", 100 /* max requests */, 60 * 1000 /* windowMs */);
    }
}
```

#### Step 3: Apply to a REST Endpoint

```java
@RestController
public class UserController {

    @GetMapping("/api/users")
    public ResponseEntity<String> getUsers() {
        return ResponseEntity.ok("Users data");
    }
}
```

**Note**: Use `@EnableRateLimit` on your controller or configure the rate limiter globally via Spring Cloud Gateway.

---

### **GraphQL Example: Apollo Server + Rate Limiting**

In GraphQL, apply rate limiting to the `/graphql` endpoint using a custom filter.

#### Step 1: Set Up GraphQL Endpoint

```java
@Configuration
public class GraphQLConfig {

    @Bean
    public GraphQL graphQL() {
        // Configure your schema here
        return new GraphQL(...);
    }
}
```

#### Step 2: Apply Rate Limiting

```java
@Configuration
public class RateLimitConfig {

    @Bean
    public FilterRegistrationBean<GraphQLRateLimiter> graphqlRateLimiter() {
        FilterRegistrationBean<GraphQLRateLimiter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new GraphQLRateLimiter());
        registration.addUrlPattern("/graphql");
        return registration;
    }
}
```

**Key Consideration**: Use IP-based or token-based rate limits for GraphQL to avoid overloading the server.

---

## 2. Caching: REST vs GraphQL in Spring Boot

### What is Caching?

Caching stores responses to reduce redundant processing and improve performance.

---

### **REST Example: Spring `@Cacheable` Annotation**

Use the `@Cacheable` annotation for in-memory caching.

```java
@Service
public class UserService {

    @Cacheable("users")
    public List<User> getUsers() {
        // Fetch data from DB
        return userService.findUsers();
    }
}
```

**Configuration**:

```properties
spring.cache.type=redis
```

---

### **GraphQL Example: Apollo Cache Control**

For GraphQL, use Apollo's `@CacheControl` or Redis for query-level caching.

#### Step 1: Enable Redis Caching

```properties
spring.data.redis.host=localhost
spring.data.redis.port=6379
```

#### Step 2: Use Redis Cache with Apollo Server

```java
@Configuration
public class CacheConfig implements GraphQLCache {

    @Override
    public GraphQL buildGraphQL() {
        return new ApolloServer(...)
            .withCache(new RedisCache("graphql-cache"))
            .build();
    }
}
```

**Key Consideration**: Cache GraphQL queries by their string representation and variables to avoid stale data.

---

## 3. Throttling: REST vs GraphQL in Spring Boot

### What is Throttling?

Throttling limits request rate and delays or denies excessive requests to protect against DDoS attacks.

---

### **REST Example: Spring Throttling**

Use `spring-rate-limit` with a stricter threshold.

```java
@Configuration
public class ThrottleConfig {

    @Bean
    public RateLimiter throttleLimiter() {
        return new RateLimiter("throttleApp", 50 /* max requests */, 60 * 1000 /* windowMs */);
    }
}
```

Apply it to endpoints using `@EnableThrottle`.

---

### **GraphQL Example: Throttling with Filters**

Apply the same throttler to GraphQL endpoints as shown earlier.

```java
@Configuration
public class ThrottleConfig {

    @Bean
    public FilterRegistrationBean<GraphQLThrottler> graphqlThrottler() {
        FilterRegistrationBean<GraphQLThrottler> registration = new FilterRegistrationBean<>();
        registration.setFilter(new GraphQLThrottler());
        registration.addUrlPattern("/graphql");
        return registration;
    }
}
```

**Note**: Use session or token-based throttling for GraphQL to differentiate between users.

---

## Best Practices Summary

| Concept           | REST                         | GraphQL                              |
| ----------------- | ---------------------------- | ------------------------------------ |
| **Rate Limiting** | `@EnableRateLimit` + Gateway | Apply limiter to `/graphql` endpoint |
| **Caching**       | `@Cacheable` + Redis         | Apollo cache control or Redis        |
| **Throttling**    | `RateLimiter` with IP/token  | Throttle GraphQL endpoint            |

---

## Conclusion

While REST and GraphQL have different structures, the core principles of rate limiting, caching, and throttling remain similar. In **Spring Boot**, REST APIs leverage annotations like `@Cacheable` and custom filters, while GraphQL benefits from Apollo Server integrations or Redis-based caching. By understanding these differences, you can build **secure**, **scalable**, and **high-performance** APIs in both paradigms.

---

### Tools & Libraries for Spring Boot

- **Rate Limiting**: `spring-rate-limit`, [Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway)
- **Caching**: `@Cacheable`, [Spring Data Redis](https://spring.io/projects/spring-data-redis)
- **GraphQL**: [Spring GraphQL](https://spring.io/projects/spring-graphql), [Apollo Server](https://www.apollographql.com/docs/apollo-server/)

---

**🚀 Let’s build something amazing! If you have a project in mind or need help with your next design system, feel free to reach out.**  
📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀

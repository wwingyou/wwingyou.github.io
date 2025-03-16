---
id: 1740270399-summary-spring-security-servlet-applications-getting-started
aliases:
  - summary-spring-security-servlet-applications-getting-started
tags: SpringSecurity
layout: post
---

[Docs link](https://docs.spring.io/spring-security/reference/servlet/getting-started.html)

## Summary

- When you include Spring Security in your Spring Boot projects, it does the following configuration:
```java
@EnableWebSecurity 
@Configuration
public class DefaultSecurityConfig {
    @Bean
    @ConditionalOnMissingBean(UserDetailsService.class)
    InMemoryUserDetailsManager inMemoryUserDetailsManager() { 
        String generatedPassword = // ...;
        return new InMemoryUserDetailsManager(User.withUsername("user")
                .password(generatedPassword).roles("USER").build());
    }

    @Bean
    @ConditionalOnMissingBean(AuthenticationEventPublisher.class)
    DefaultAuthenticationEventPublisher defaultAuthenticationEventPublisher(ApplicationEventPublisher delegate) { 
        return new DefaultAuthenticationEventPublisher(delegate);
    }
}
```
- `@EnableWebSecurity` annotation publishes Spring Security's defualt `Filter` chain as a `@Bean`.
- `InMemoryUserDetailsManager` and `DefaultAuthenticationEventPublisher` are published with `@ConditionalOnMissingBean` annotation, allowing them to serve as fallbacks.

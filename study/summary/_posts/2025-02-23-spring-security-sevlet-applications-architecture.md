---
id: 1740272921-summary-spring-security-sevlet-applications-architecture
aliases:
  - summary-spring-security-sevlet-applications-architecture
tags: SpringSecurity
layout: post
---

[Docs link](https://docs.spring.io/spring-security/reference/servlet/architecture.html)

## Summary

- Spring Security's Servlet support is based on Servlet Filter.
- A `Filter` can prevent a downstream `Filter` or `Servlet` from being invoked by a `FilterChain` instance.
- A `Filter` can modify the `HttpServletRequest` and `HttpServletResponse` before they are used by a downstream `Filter` or `Servlet`.
- Spring uses `DelegatingFilterProxy` to integrate Servlet filter chain with spring framework's `ApplicationContext`.
- `DelegatingFilterProxy` delegates all the works to the spring Beans that implements `Filter`.
- Spring Security's Servlet support is provided by the `FilterChainProxy`.
- `FilterChainProxy` is a Bean, which means it is wrapped with the `DelegatingFilterProxy`.
- The `SecurityFilterChain` is used by `FilterChainProxy` to determine which `Filter` Bean should be invoked for the current request.
- `SecurityFilterChain`s are indepenent of each other.
- Security Filters has specific order to be invoked. You can check the exact order in the [FilterOrderRegistration code.](https://github.com/spring-projects/spring-security/blob/6.4.3/config/src/main/java/org/springframework/security/config/annotation/web/builders/FilterOrderRegistration.java)
- To see the log of Security Filter's invokation, set following properties in `application.properites`:
```properties
logging.level.org.springframework.security=TRACE
```
- Most of the time, defualt Security Filters are enough to serve security interests. But, if publishing custom `Filter` into `SecurityFilterChain` is needed, use `#addFilterBefore(Filter, Class<?>)`, `#addFilterAfter(Filter, Class<?>)`, `#addFilterAt(Filter, Class<?>)` of `HttpSecurity` to insert or replace custom Filters into specific place.
- To determine the location of custom filter, take a look at these following events:
    1. `SecurityContext` is loaded from the session
    2. Request is protected from common exploits; **secure headers**, **CORS**, **CSRF**
    3. Request is **authenticated**
    4. Request is **authorized**
- If you want to add custom authenetication filter, place it after `LogoutFilter`.
- The other recommanded insertion points are described [here.](https://docs.spring.io/spring-security/reference/servlet/architecture.html#_adding_a_custom_filter)
- Instead of extending `Filter` and registering it into Security Filter Chain, you can extend `OncePerRequestFilter` as a Spring Bean. This ensures that the filter will be invoked once per every request.
- Custom Security Filters should not be a Spring Bean since it will automatically be registered into Servlet Filter Chain. However, if you really need to make it as a Spring Bean, check [here](https://docs.spring.io/spring-security/reference/servlet/architecture.html#_declaring_your_filter_as_a_bean) to see how to prevent them from being registered automatically.
- `ExceptionTranslationFilter` is inserted into the `SecurityFilterChain` as one of the Securit Filters.
- It invokes downstream filter first. If an exception occurs, it translate the exception into proper HTTP response (redirect to the login page or return error response).
- To re-request the previous on after user is authenticated, Spring Security uses `RequestCache` to store previous request.
- You can customize `RequestCache` when to store request or not.
- If there's store cache in `RequestCache` instance, `RequestCacheAwareFilter` replay the original request.

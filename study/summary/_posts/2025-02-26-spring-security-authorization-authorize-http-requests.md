---
id: 1740559127-summary-spring-security-servlet-applications-authorization-authorize-http-requests
aliases:
  - summary-spring-security-servlet-applications-authorization-authorize-http-requests
tags: SpringSecurity
layout: post
---

[Docs link](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html)

## Summary

- Spring Security allow you to model authentication at the request level.
- `AuthorizationFilter` is the core executor to make a access decision.
- The `AuthorizationFilter` passes the `Supplier<Authentication>` and the `HttpServletRequest` to the `AuthenticationManager`, which is typically the `RequestMatcherDelegatingAuthenticationManager`.
- Authorization in Spring Security is not only applied to all requests, but all dispatches.

---
id: 1740394012-summary-spring-security-servlet-applications-authentication-persistence
aliases:
  - summary-spring-security-servlet-applications-authentication-persistence
tags: SpringSecurity
layout: post
---

[Docs link](https://docs.spring.io/spring-security/reference/servlet/authentication/persistence.html)

## Summary

- Spring Security associates user to futer request by `SecurityContextRepository` interface.
- The default implementation of `SecurityContextRepository` is `DelegatingSecurityContextRepository`, which delegates to `HttpSessionSecurityContextRepository` and `RequestAttributeSecurityContextRepository`.
- `SecurityContextHolderFilter` loads `SecurityContext` from the `SecurityContextRepository` and save it to the `SecurityContextHolder`.
- When the `SecurityContext` changed, you have to manually store it to the `SecurityContextRepository` as well as `SecurityContextHolder`.

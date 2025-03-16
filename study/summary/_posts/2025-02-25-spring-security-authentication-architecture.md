---
id: 1740462603-summary-spring-security-servlet-applications-authentication-authentication-architecture
aliases:
  - summary-spring-security-servlet-applications-authentication-authentication-architecture
tags: SpringSecurity
layout: post
---

[Docs link](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authentication)

## Summary

- `SecurityContextHolder` is the heart of Spring Security's authentication model, which contains the details of user who is authenticated.
- The simplest way to indicate a user is authenticated is to set the `SecurityContextHolder` directly.
- You can obtain `Authentication` from `SecurityContext`.
- `Authentication` serves two main purposes within Spring securiy.
- One is the container of user credentails to be passed to the `AuthenticationManager`.
- The other is the container of currently authenticated user information.
- You can determine the state of a `Authentication` instance using `isAuthenticated()` method.
- `GrantedAuthority` represents an authority that is granted to the principal.
- `AuthenticationManager` is the API defines how Spring Security's Filters perform authentication.
- `ProviderManager` is the most common `AuthenticationManager` implementation which delegates to a list of `AuthenticationProvider`.
- Each `AuthenticationProvider` has an opportunity to authenticate.
- If an `AuthenticationProvider` fails to authenticate, it passes the opportunity to the downstream `AuthenticationProvider`.
- Here's two most common `AuthenticationProvider`s:
    1. `DaoAuthenticationProvider` for username/password authentication
    2. `JwtAuthenticationProvider` for JWT token authentication
- When user doesn't provide proper authentication information for the requested resource, `AuthenticationEntryPoint` provides entry point of authentication (e.g. redirect to the login page, respond with an **WWW-Authenticate** header, or else).
- `AbstractAuthenticationProcessingFilter` is the base `Filter` for authenticating user credentails.
- The `AbstractAuthenticationProcessingFilter` implementation is responsible for extracting `Authentication` from the request and validating it by passing it to the `AuthenticationManager`.

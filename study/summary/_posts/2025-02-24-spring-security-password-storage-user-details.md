---
id: 1740374104-summary-spring-security-servlet-applications-authentication-username-password-password-storage-user-details
aliases:
  - summary-spring-security-servlet-applications-authentication-username-password-password-storage-user-details
tags: SpringSecurity
layout: post
---

[Docs link](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/user-details.html)

## Summary

- `UserDetails` is returned by the `UserDetailsService`.
- `DaoAuthenticationProvider` validates the `UserDetails` and returns an `Authentication` that has a principal that is the `UserDetails` returned by the configured `UserDetailsService`.
- Implementing `CredentialsContainer` interface in classes containing user credentials is highly recommanded.
- `CredentialsContainer` interface provides `#eraseCredentials()` method, which clears user credentials after the authentication is completed.

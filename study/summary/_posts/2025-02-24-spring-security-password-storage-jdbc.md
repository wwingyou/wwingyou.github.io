---
id: 1740371577-summary-spring-security-servlet-application-authentication-uasername-password-password-storage-jdbc
aliases:
  - summary-spring-security-servlet-application-authentication-uasername-password-password-storage-jdbc
tags: SpringSecurity
layout: post
---

[Docs link](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/jdbc.html)

## Summary

- `JdbcDaoImpl` implements `UserDetailsService` to provide support for username password authentication that is retrieved by using JDBC.
- `JdbcUserDetailsManager` extends `JdbcDaoImpl` to provide management of `UserDetails` through `UserDetailsManager` interface.

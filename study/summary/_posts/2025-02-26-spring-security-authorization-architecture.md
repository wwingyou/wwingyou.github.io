---
id: 1740491333-summary-spring-security-servlet-applications-authorization-authorization-architecture
aliases:
  - summary-spring-security-servlet-applications-authorization-authorization-architecture
tags: SpringSecurity
layout: post
---

[Docs link](https://docs.spring.io/spring-security/reference/servlet/authorization/architecture.html)

## Summary

- The authority in Spring Security is represented by `GrantedAuthority` interface.
- The `GrantedAuthority` has only one method, `#getAuthority()`, which returns representation of authority as a `String`.
- `GrantedAuthority` is stored in `Authentication` instance by the `AuthenticationManager`, then used by the `AccessDesicionManager` when making authorization decisions.
- When the authority is complicated enough to represented as a `String`, `#getAuthority()` should return `null`.
- Then, `AuthorizationManager` should deal with the specialized authority.
- There's only one `GrantedAuthority` that Spring Security provide: `SimpleGrantedAuthority`.
- By default, role-based authorization rules include `ROLE_` as a prefix.
- `AuthorizationManager` is responsible for both pre-authorization and post-authorization to secure objects.
- The method authorization process in Spring Security is done by the interceptors, typically `AuthorizationManagerBeforeMethodInteceptor` and `AuthorizationManagerAfterMethodInterceptor`, which utilize `AuthorizationManager` to control the access and secure the object.
- `AuthorityAuthorizationManager` is the most common `AuthorizationManager`, which makes authority decision by checking whether there're any matching authority stored in `Authentication` object.
- If you need a hierarchical role, use `RoleHierarchy` to define role hierarchy as following:

```java
@Bean
static RoleHierarchy roleHierarchy() {
    return RoleHierarchyImpl.withDefaultRolePrefix()
        .role("ADMIN").implies("STAFF")
        .role("STAFF").implies("USER")
        .role("USER").implies("GUEST")
        .build();
}

// and, if using pre-post method security also add
@Bean
static MethodSecurityExpressionHandler methodSecurityExpressionHandler(RoleHierarchy roleHierarchy) {
	DefaultMethodSecurityExpressionHandler expressionHandler = new DefaultMethodSecurityExpressionHandler();
	expressionHandler.setRoleHierarchy(roleHierarchy);
	return expressionHandler;
}
```

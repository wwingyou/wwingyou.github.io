---
id: 1740199127-summary-spring-security-features-authentication-password-storage
aliases:
  - summary-spring-security-features-authentication-password-storage
tags: SpringSecurity
layout: post
---

[Docs link](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html)

## Summary

- Spring security provide `PasswordEncoder` to perform a one-way transformation of password to store it securely.
- Storing passwords in plaintext poses a risk of being exposed due to SQL injection.
- It is better to store hashed passwords instead of plaintext to make it harder for malicious users to obtain the origianal value.
- In modern times, cryptographic hashes (like SHA-256) are no longer secure since hardware power increased over time.
- Now we use adaptive one-way hashes, which are designed to adjust the level of resource-intensiveness using a 'work factor' (bcrypt, PBKDF2, etc...)
- Since adaptive one-way hashes intentionally degrade application performance, users are encouraged to exchange long-term credentials (such as username and password) for a short-term credential (such as session or OAuth token).
- The short term credentials can be validated quickly without any loss of security.
- Instead of using a concrete `PasswordEncoder`, Spring Security uses `DelegatingPasswordEncoder` by default to support different password encoders.
- `DelegatingPasswordEncoder` stores IDs indicating which `PasswordEncoder` is used, as a prefix to the stored value (the format is `{id}encodedPassword`).
- You can build a `UserDetails` with default `PasswordEncoder` like this:
```java
User.withDefaultPasswordEncoder()
    .username("username")
    .password("password")
    .roles("USER", "ADMIN")
    .build();
```
- However, creating user credentials this way is discouraged, since the passwords can be exposed in the source code and compiled class files.
- Instead, generate hashed password externally and store it manually. A common approach is to use Spring CLI:
```
$ spring encodepassword password
{bcrypt}$2a$10$X5wFBtLrL/kHcmrOGGTrGufsBX8CJ0WpQpF3pgeuxBB/H73BK1DW6
```
- Spring Security provide functionality to redirect well-known change password
url (`/.well-known/change-password`) to the actual password change endpoint.
- The configuration below redirect `/.well-known/change-password` to `/change-password`:
```java
http
    .passwordManagement(Customizer.withDefaults())
```
- You can specify the change password redirect url like this:
```java
http
    .passwordManagement((management) -> management
        .changePasswordPage("/update-password")
    )
```
- Spring Security also provides functionality to check whether the password has been compromised or not. A detailed explanation can be found [here.](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html#authentication-compromised-password-check)

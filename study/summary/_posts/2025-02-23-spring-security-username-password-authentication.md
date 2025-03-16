---
id: 1740296892-summary-spring-security-servlet-applications-authentication-username-password
aliases:
  - summary-spring-security-servlet-applications-authentication-username-password
tags: SpringSecurity
layout: post
---

[Docs link](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/index.html#servlet-authentication-unpwd)

## Summary

- You can configure username and password authentication using the following:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

	@Bean
	public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
		http
			.authorizeHttpRequests((authorize) -> authorize
				.anyRequest().authenticated()
			)
			.httpBasic(Customizer.withDefaults())
			.formLogin(Customizer.withDefaults());

		return http.build();
	}

	@Bean
	public UserDetailsService userDetailsService() {
		UserDetails userDetails = User.withDefaultPasswordEncoder()
			.username("user")
			.password("password")
			.roles("USER")
			.build();

		return new InMemoryUserDetailsManager(userDetails);
	}

}
```
- The above configuration registers `DaoAuthenticationProvider`, which uses `UserDetailsService` and `PasswordEncoder` to authenticate user, and enables **Form Login** and **HTTP Basic** authenticaion.
- Customizing `AuthenticationManager` can be achieved in two ways.
- One is to use `AuthenticationManagerBuilder` which is published as a bean, while the other is to override the default `AuthenticationManager` with a custom one.

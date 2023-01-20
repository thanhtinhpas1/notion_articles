# Spring Security

## Authentication and Access Control

Application security boils down to two more or less independent problems: **authentication** (who are you?) and **authorization** (what are you allowed to do?). Sometimes people say “**access control”** instead of “authorization”, which can get confusing, but it can be helpful to think of it that way because “authorization” is overloaded in other places. Spring Security has an architecture that is designed to separate authentication from authorization and has strategies and extension points for both.

### Authentication

The main strategy interface for authentication  is `AuthenticationManager` , which has only one method:

```java
public interface AuthenticationManager {
	Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```

An `AuthenticationManager` can do one of 3 things in its `authenticate()` method:

- Return an `Authentication` (normally with `authenticated=true` if it can verify that the input represents a valid principal.
- Throw an `AuthenticationException` if it believes that the input represents an invalid principal.
- Return `null` if it cannot decide.

`AuthenticationException` is a runtime exception. It is usually handled by application in a generic way, depending on the style or purpose of the application. In other worlds, user code is not normally expected to catch and handle it. For example, a web UI might render a page that says that the authentication failed, and a backend HTTP service might send a 401 response, with or without a `WWW-Authenticate` header depending on the context.

The most commonly used implementation of `AuthenticationManager` is `ProviderManager`, which delegates to a chain of `AuthenticationProvider` instances. An `AuthenticationProvider` is a bit like an `AuthenticationManager` , but it has an extra method to allow the caller to query whether it supports a given `Authentication` type:

```java
public interface AuthenticationProvider {
	Authentication authenticate(Authentication authentication) throws AuthenticationException;
	boolean supports(Class<?> authentication);
}
```

The `Class<?>` argument in the `supports()` method is really `Class<? extends Authentication` (it is only ever asked if it supports something that is passed into the `authentication()` method). A `ProviderManager` can support multiple different authentication mechanism in the same application by delegating to a chain of `AuthenticationProviders.` If a `ProviderManager` does not recognize a particular `Authentication` instance type, it is skipped.

A `ProviderManager` has an option parent, which it can consult if all providers return `null`. If the parent is not available, a `null Authentication` results in an `AuthenticationException`.

Sometimes, an application has logical groups of protected resources (for example, all web resources that match a path pattern, such as `/api/**` ), and each group can have its own dedicated `AuthenticationManager`. Often, each of those is a `ProviderManager` , and they share a parent. The parent is then a kind of `global` resource, acting as a fallback for all providers.

![Untitled](Spring%20Security%20c93d76c009854a418ba09efeb72906b0/Untitled.png)

### Customizing Authentication Managers

Spring Security provides some configuration helpers to quickly get common authentication manager features set up in your application. The most commonly used helper is the `AuthenticationManagerBuilder` which is great for setting up in-memory, JDBC, or LDAP user details or for adding a custom `UserDetailsService`. The following example shows an application that configures the global (parent) `AuthenticationManager` :

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {
 // web stuff here

 @Autowired
 public void initialize(AuthenicationManageBuilder builder, DataSource dataSource) {
  builder.jdbcAuthentication().dataSource(dataSource).withUser("dave").password("secret").roles("USE");
 }
}
```

This example relates to a web application, but the usage of `AuthenticationManagerBuilder` is `@Autowired` into a method in a `@Bean` - that is what makes it build the global (parent) `AuthenticationManager`. In contrast, consider the following example:

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {
 @Autowired
 DataSource dataSource;

 // web stuff here
  
 @Override
 public void configure(AuthenticationManagerBuilder builder) {
   builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
   .password("secret").roles("USER");
 }
}
```

If we had used an `@Override` of a method in the configurer, the `AuthenticationManagerBuilder` would be used only to build a “local” `AuthenticationManager` , which would be a child of the global one
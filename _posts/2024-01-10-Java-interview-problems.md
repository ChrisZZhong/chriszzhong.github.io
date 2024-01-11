---
layout: post
title: "Java interview questions"
date: 2023-12-28
description: "frequent questions asked in interviews"
tag: Interviews
---

## Lambda expression

lambda expression is used to create an anonymous function. (para) -> {action}

The usage is in Java 8, we can use `lambda expression` to initialize a `functional interface`.

## Functional Interface

An interface that has only one abstract method and provides a single functionality. `@FunctionalInterface` annotation is not mandatory but it will add constraints to check if there is only one method in it.

- `Consumer` takes a single input and performs action without returning the result.

- `Supplier` used to generate a value.

- `Function` takes a single input and produces a result.

- `Predicate` takes a single input and returns a boolean value.

## Optional

Optional object is used to handle NPE problems, handling values as ‘availableʼ or ‘not availableʼ instead of checking null values.

```
empty(): Returns an empty Optional object
of(): Returns an Optional with the specified present non-null value
ofNullable(): Returns an Optional with the specified value, if non-null, otherwise returns an empty Optional
isPresent(): Returns true if there is a value present, otherwise false
```

```Java
Optional result = xxxService.findById(id);
if (result.isPresent()) {
    return result.get();
} else {
    throw new Exception();
}
// for service
public Optional<xxx> findById(Long id) {
    return Optional.ofNullable(xxxRepository.findById(id));
}
```

## Stream API (Intermediate & Terminal Operations)

Stream API is used to process collections of objects, it is a pipeline that performs a series of operations to the object to get the desired result.

- `Intermediate` operation is lazily executed and returns a stream as a result, thus can be pipelined. `filter()` , `map()` , `flatMap()` , `distinct()` , and `sorted()`.

  flatMap() : It can transform data or flatten data, like flatten a 2d array to an array.

- `Terminal` operation is eagerly executed and returns a non-stream result, thus ending the stream. `forEach()` , `count()` , `collect()` , `reduce()` , `allMatch()` , `anyMatch()` , `noneMatch()` , `findFirst()` , `findAny()`.

## Comparable & Comparator

They both are used for comparing two objects. They serve for different purposes.

- `Comparable` is used to sort objects in a natural order.

  1. Need to `implement` the `Comparable` interface and override the `compareTo()` method.

- `Comparator` is a functional interface used to sort objects in a custom order.
  1. We don't need to implement the `Comparator` interface, but we need to create a new class to implement the `Comparator` interface and override the `compare()` method.
  2. We can also use lambda expression to create a `Comparator` object.

## Spring Framework (IOC)

Core feature is **`Dependency Injection (IOC)`** - Invert the control of creating objects to the framework.

### IOC Container types

- `BeanFactory` : lazy loaded, XML
- `ApplicationContext` : eager loaded, XML & annotation

### IOC Container life cycle

- IOC container `instantiate` when spring application starts
- Read the `bean` class from config files such as XML or annotations like @ComponentScan
- `Create` bean instance based on the Bean definition
- inject the Bean when required
- Post-processor `callback` methods, postProcessBeforeInitialization and postProcessAfterInitialization are called for monitor works or proxying.

### Bean & Bean scopes

Bean is object managed by Spring IOC container. use `@Scope` to specify the scope of the bean.

- `Singleton` : `default scope` only created once during Spring application run time
- `Prototype` : new instance per request
- `Request` : new instance per HTTP request
- `Session` : new instance per HTTP session
- `GlobalSession` : new instance per global HTTP session

### Dependency Injection (Ambiguous)

- `Constructor` : `@Autowired` on the constructor (inject immutable class)
- `Setter` : `@Autowired` on the setter method (when circular dependency exist)
- `Field` : `@Autowired` on the field (tight coupling, not recommended)

If there are multiple beans that implement the same interface, the container will not know which one to inject, We can use `@Primary` to determine which bean has higher priority, or use `@Qualifier` to specify an exact bean

## Spring Boot (auto configuration)

`Spring Boot`'s core feature is **`Auto Configuration`**, which is used to reduce the time for setting up configurations.

1. `Pom` file, spring Boot will generate default dependencies for us like `security`, `web`, `JDBC`.

2. `application.properties` file, we can `override` the default configuration by providing the `key-value` pairs in the application.properties file. like the data source url, username, password.

3. `@SpringBootApplication`

   - `@EnableAutoConfiguration` : Allows auto configuration based on the .jar dependencies in the pom.xml file

   - `@ComponentScan` : scan the `@Component` and their child class like @Controller, @Repository under the basic folder.

   - `@Configuration` : Spring Boot will only scan @Bean third party class under the @Configuration class. This is used to add extra beans to the spring container.

## Spring MVC

1. DispatcherServlet (central servlet) `map the request` to the corresponding controller method.

2. Data is processed In controller, service, dao, and then return to the controller.

3. Controller create `model object` contains the data in a `key-value` pair way. Controller will return a string of the view name to the `view resolver`.

4. View resolver has two responsibilities

   - `prefix` and `suffix` to the view name to find the jsp page

   - reach the template to replace the variables with the data in the model object.

## Spring AOP

AOP is used to **decouple cross-cutting concerns** from the core business logic. (e.g. logging, exception handling) **using `proxy` to wrap the object being called.** intercept the calls and add extra functionality before or after the method call.

- `Join point` **it is the `unit of point` can be monitored.**

- `Pointcut` is a predicate or expression that matches join points. It is a `set of join points` where `advice` should be executed.

- `Advice` is the `action` executed on join points. Different types - `before`, `after`, `after-returning`, `after-throwing`, and `around`.

- `Aspect` is a module that encapsulates join points, pointcuts, and advice. It is a class that implements cross-cutting concerns.

## Spring Security

Framework to protect our endpoints.

1. Login in with username and password:

- UserDetailsService `loadUserByUsername()` to load user information from database, and then compare the credentials.

- AuthenticationManager `authenticate()` to authenticate the user and generate `UserDetails` object with GrantedAuthority.

- Generate token with UserDetails.

2. user want to access the endpoint with token: first it will be filtered by the filter chain.

- JwtTokenUtil resolve() the token from request and generate the `userDetails` object.

- generate authentication object and set the authentication object to SecurityContext.

For authroization, we can use `@PreAuthorize("hasRole('ROLE_ADMIN')")` or `@PreAuthorize("hasAuthority('unverified')")` to check the role of the user. Also, we can use `@AuthenticationPrincipal UserDetails userDetails` to get the user details in the controller.

## JWT (JSON Web Token)

JWT is a standard that allows you to encode data in a JSON format. It is often used to encode authentication information, like the username, the permissions, etc. It is often used in combination with OAuth2.

JWT mainly contains three parts:

- Header: `Hashing algorithm` used to sign the token.

- Payload: contains the `claims` like userId, the permissions, etc.

- Signature: is used to `verify` that the `sender` of the JWT and to ensure that the message wasn’t changed along the way. (Created by hashing the header and the payload with a secret key)

## API Gateway

API Gateway is a `single entry point` for all the microservices. `Route requests` to the appropriate microservice. It is not necessary, but it is good for `decoupling` and `load balancing` (round robin).

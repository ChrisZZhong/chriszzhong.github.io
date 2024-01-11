---
layout: post
title: "Java interview questions"
date: 2023-12-28
description: "frequent questions asked in interviews"
tag: Interviews
---

## [project story](https://chriszzhong.github.io/2022/09/script/)

## Immutability

state of an object that cannot be changed after it is created.

1. final variable: value (reference) cannot be changed once assigned.
2. final method: cannot be overridden.
3. final class: cannot be extended by other class.

### How to create Immutable Class

1. Declare the class as final so it can’t be extended.
2. Make all fields private and final.
3. No setter methods. All variables are set in the constructor with the deep copy of the input.
4. Return copies of feild data in getter methods.

### String is immutable

1. `Security concern`: sensitive information such as password, network connection, etc. Being immutable prevents malicious code from modifuing sensitive information.

2. `Thread safe`: Immutable objects are by default thread safe, can be shared among multiple threads.

3. `Hashcode`: String is widely used as key in Map and other collection classes. Being immutable guarantees that hashcode will always be same so that it can be cached without worrying about the changes.

4. `String Pool`: Save the runtime memory by reusing strings.

## Thread

1. `New`: new thread.

2. `Runnable`: call `start()` method, ready to run.

3. `Running`: When the thread is running

4. `Blocked`: The thread will be in blocked state when it is trying to acquire a lock but currently the lock is acquired by the other thread. The thread will move from the blocked state to runnable state when it acquires the lock.

5. `Waiting`: When we call the `wait()`, `join()` method, the thread will be in the waiting state and `release the lock` until another thread calls the `notify()` or `notifyAll()` method.

6. `Timed Waiting`: When we call the `sleep()` method, the thread will be in the timed waiting state and `release the lock` for the specified time. When the time is up, the thread will move to the runnable state.

7. `Terminated`: When the thread finishes its execution, it will be in the terminated state.

### Create a thread

Generally, we can create a thread in two ways: Implement `Runnable` interface or extend `Thread` class.

But we use Runnable interface instead of Thread class, because:

1.  Thread `implements` runnable, when extending Thread, we only override run() method. this is a violation of `IS-A` principle.

2.  `After extending` Thread class, we can't extend other classes.

Example:

```Java
Runnable task = () -> someReallyLongProcess();
Executor executor = Executors.newSingleThreadExecutor();
executor.execute(task);
```

We need to access the result and when bug happens, we need to know the message of the exception. But `Runnable` can not `return the result` and `throw exception`, so we can use `Callable` and `Future` to solve this problem.

### Callable

`Callable` is a functional interface, which has only one method call(). Callable is similar to Runnable, but Callable can return the result and throw exception.

```Java
@FunctionalInterface
public interface Callable<V> {

   V call() throws Exception;

}
```

`Executor` only accepts `Runnable`, so we need to use `ExecutorService` to run the `Callable` thread.

```Java
public interface ExecutorService extends Executor {

   // Submits a value-returning task for execution and returns a Future representing the pending results of the task.
   <T> Future<T> submit(Callable<T> task);

   // Submits a Runnable task for execution and returns a Future representing that task.
   Future<?> submit(Runnable task);

   // Submits a Runnable task for execution and returns a Future representing that task.
   <T> Future<T> submit(Runnable task, T result);
}
```

### Future

The `Future` represents the result of Thread. When thread executed, the result or the exception message will be stored in the Future for main thread to access.

```Java
// In the main thread
Callable<String> task = () -> buildPatientReport();
Future<String> future = executor.submit(task);

String result = future.get();   // blocking here until finished to get the result
```

**`DownSide`**: Threads should be independent, also we need multi-threads to improve the performance, but `Future` will `block` the main thread, so we need to use CompletableFuture to solve this problem.

### CompletableFuture

`CompletableFuture` is almost the same as a Future object with more methods and more capabilities. (`runnable`, `supplier`)

```java
Runnable task = () -> System.out.println("Hello world");
ExecutorService service = Executors.newSingleThreadExecutor();
Future<?> future = service.submit(task);    // use future object
CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(task);
```

`CompletableFuture` do not support `Callable`, only support `Runnable` and `Supplier`.

```java
Supplier<String> task = () -> readPage("https://google.com");
ExecutorService service = Executors.newSingleThreadExecutor();
CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(task);
```

Future chain to handle the results:

```java
CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(task);
CompletableFuture<Integer> lengthFuture = completableFuture.thenApply(String::length).thenApply(len -> len * 2).exceptionally(ex -> 0);
```

### ExecutorService

`ExecutorService` is a thread pool used to manage threads, and can submit tasks to the thread pool for execution. Generally, we can use `ExecutorService` to run Runnable, Callable, CompletableFuture threads.

Advantage:

- Easy to manage threads and reuse threads, which can reduce the overhead of thread creation and improve performance.

- Can limit the number of threads, avoid too many threads to cause system crash.

Disadvantage:

- A careless threads pool size can halt the system and bring performance down

- Always shut down the executor service after tasks are completed and service is no lon..ger needed. Otherwise, JVM will never terminate.

## JVM

Two systems: class loader, Execution Engine
Two components: `Runtime data area`, `native interface`

- `Class loader`: loading classes into memory
- `Execution Engine`: JIT compiler, compiles bytecode to machine code

- `Runtime data area`:
  `Method area` used to store class-level structures and metadata, and share memory with all threads in JVM. (callstacks)
  `Heap`:store objects, divided into the `Young Generation`, `Old Generation` (Tenured), MetaSpace. GC runs here.

Native interface:
allows Java code to interact with native libraries written in other languages, such as C and C++.

### JVM Heap

- `Young Generation`: store newly created objects

- `Old Generation`: store long-lived objects, Major GC runs here.

- `MetaSpace`: store class metadata about classes, methods, and other runtime structures, it can dynamically grow and shrink as needed.

# Java 8

## Lambda expression

lambda expression is used to create an anonymous function. (para) -> {action}

The usage is in Java 8, we can use `lambda expression` to initialize a `functional interface`. Like `Comparator` interface, we can use lambda expression to create a `Comparator` object when using priority queue.

## Functional Interface

An interface that has only one abstract method and provides a single functionality. `@FunctionalInterface` annotation is not mandatory but it will add constraints to check if there is only one method in it.

- `Consumer` takes a single input and performs action without returning the result.
- `Supplier` used to generate a value.
- `Function` takes a single input and produces a result.
- `Predicate` takes a single input and returns a boolean value.

## Optional

Optional object is used to handle NPE problems, handling values as ‘availableʼ or ‘not availableʼ instead of checking null values.

- empty(): Returns an empty Optional object
- of(): Returns an `non-null` value or throws `NullPointerException` if null
- ofNullable(): Returns an Optional with the specified value, if non-null, otherwise returns an empty Optional
- isPresent(): Returns true if there is a value present, otherwise false
- get(): Returns the value if present, otherwise throws NoSuchElementException

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

## Default & Static methods in Interface

- `Default` add a new method to the interface without modifying implementation classes, like Stream().

- `Static` enable to define a method belonging to the interface itself. It cannot be overridden by implementation classes.

# Spring

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

## Cache - (stale data)

Problem: When we update the data in the database, the cache is not updated. So the data in the cache is stale.

- Set expiration times for cached items.
- Use `@CacheEvict` to remove the cache when updating the data.
- Use `@CachePut` to update the cache when updating the data.

# Microservices

## API Gateway

API Gateway is a `single entry point` for all the microservices. `Route requests` to the appropriate microservice. It is not necessary, but it is good for `decoupling` and `load balancing` (round robin).

## Microservices Debug (Logging & Distributed Tracing)

- Centralized logging: `ELK` (Elasticsearch, Logstash, Kibana) to centralize the logs from different microservices.

- Distributed tracing: `Zipkin`, use Zipkin to trace the request from the client to the microservices.

When Bug happens, we can use the `traceId` to trace the request from the client to the microservices. Then we can use the `traceId` to search the logs in the ELK to find the bug.

## Kafka

Kafka is a distributed streaming platform that is designed for high-throughput, and real-time data processing.

**Broker, Topic, Partition, Producer, Consumer, Consumer group, zookeeper**

[link](https://chriszzhong.github.io/2023/06/kafka/)

## Fault-Tolerance

Enables system to continue `works` though there are some `failures` inside of it.

The more faults a system can tolerate without failing, the more fault-tolerant it is.

- `Circuit breakers` can prevent repeated requests to a failing service, allowing it time to recover.

- Timeouts can prevent a service from waiting too long for a response from another service. Fail fast is better than waiting indefinitely.

- `load balancers` : `avoid routing traffic` to unhealthy instances.

## RESTFul API 6 principles

1. `Uniform Interface` : requests looks same

2. `Client-Server decoupling` : client and server `independent`of each other. Client - `user interface` and server - `backend data storage`. Communicate through REST API.

3. `Stateless` : `server X store client data`. Each request contain all the information needed.

4. `Cacheability` : When possible, resources should be cacheable on the client or server side.

5. `Layered System` : Client does not need to know the internal structure (server / intermediary).

6. `Code on demand` : The server can provide executable code or scripts for the client to execute in its context.

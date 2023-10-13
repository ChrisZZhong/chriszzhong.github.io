---
layout: post
title: "Software Interview problems"
date: 2023-10-11
description: "Software Interview problems"
tag: Interviews
---

# Core Java

## Solid Principles

### Single Responsibility Principle

- **A class should have one and only one reason to change, meaning that a class should have only one job.**

  **Example:**  
  Suppose you have a shape class, like triangle, you want to have a method to give you the area of the shape, instead of writing a method called calculateArea, you should create a new class called areaCalculator which is responsible for calculating area of all types, and call areaCalculator when you need to calculate the area.

### Open-Closed Principle

- **Objects or entities should be open for extension but closed for modification.**

  **Example:**  
   you have different payment methods, like card, cash, third parties, instead of checking input several times with if else statement, you should generate a new interface like payProcessor to handle the payment, this is good for decoupling. You don’t have to modify this class when a new payment method is added, all you need to do is generate a new class and extend the payProcessor.

### Liskov Substitution Principle

- **Every subclass/derived class should be substitutable for their base/parent class.**

  **Example:**  
   List and arrayList are both list, so you can use arrayList anywhere you use list.

### Interface Segregation Principle

- **A client should never be forced to implement an interface that it doesn’t use**

  **Example:**  
   Take payment as mentioned above as an example, A and B both implement payProcessor, A have charge and refund function, suppose B do not support refund, but B have to implement an empty method, this is a violation. Instead, we should separate the parProcessor to chargeProcessor, refundProcessor.

### Dependency Inversion Principle

- **The high-level module must not depend on the low-level module, but they should depend on abstractions.**

  **Example:**  
   Still use payment as an example, if you want to use a new payment method, you should not modify the payProcessor, instead, you should create a new class and extend the payProcessor, and use the new class to handle the new payment method.

## Final, Finally, Finalize

- `final` is a keyword to declare a constant variable, a method or a class.

  1. final variable: value (reference) cannot be changed once assigned.
  2. final method: cannot be overridden.
  3. final class: cannot be extended by other class.

- `finally` is used at the end of the try-catch block to execute the code within it regardless of the result of the try-catch block.

- `finalize` is a method of Object class that is called by the garbage collector when the object is no longer referenced. Use try with resource instead, because finalize() calling is unpredictable, sometimes will cause GC pause.

## Immutable Class

**Definition**: Immutable Class is a class whose instances (objects) **cannot be modified after they are created.**

**Example**: Wrapper classes, String (Security reason, hashcode)

### How to create Immutable Class

1. Declare the class as final so it can’t be extended.

2. Make all fields private and final.

3. No setter methods. All variables are set in the constructor with the deep copy of the input.

4. Return copies of feild data in getter methods.

### Why String is immutable in Java?

1. `Security concern`: sensitive information such as password, network connection, etc. Being immutable prevents malicious code from modifuing sensitive information.

2. `Thread safe`: Immutable objects are by default thread safe, can be shared among multiple threads.

3. `Hashcode`: String is widely used as key in Map and other collection classes. Being immutable guarantees that hashcode will always be same so that it can be cached without worrying about the changes.

4. `String Pool`: Save the runtime memory by reusing strings.
   > Java uses a string pool to store and reuse string literals. When you create a string, Java checks if it already exists in the string pool. If it does, the existing instance is returned, if not, a new instance is created.

## String, StringBuffer, StringBuilder

> They both used to store string values

- `String` is immutable, used to store values with fixed size. (Security reason, Thread safe, Hashcode, String Pool)

- `StringBuffer` used when we do not need to consider the thread safety, and when we need to frequently modify the content of the string.

- `StringBuilder` used in multi-threaded environment, when we need to frequently modify the content of the string.

## HashMap

HashMap is a bucket array to store `key-value pairs`. It stores key-value pairs based on their hash value. It uses the `hashcode()` method to calculate the hash value of the key, and then uses the hash value to calculate the index of the bucket array.

When hash collision occurs, it stores the key-value pairs in the same bucket array in the form of linked list. When the number of linked lists in the same bucket array exceeds 8 or 16, it will be converted to a red-black tree to improve the search efficiency.

## Thread

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

### Callable and Future

#### Callable

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

#### Future

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

## synchronization

Control the access of multiple threads to any shared resource to achieve consistency and avoid race problem.

### Method synchronization & Block synchronization

Use `synchronized` `keyword` to achieve synchronization. Add `synchronized` to the method declaration, or add `synchronized` to the block. They will both bock other threads to access the method or block until the current thread finished and release the lock.

## Marker Interface

`Marker interface` is an `empty` interface used to provide run-time information about objects. Common examples of marker interfaces are `Serializable`.

## Serialization & Serializable

`Serialization` is convert the object to a byte stream, which can be stored in a file or sent over the network, and then be deserialized later to get the object back. The advantage of serialization is that it is easy to store and transmit data over network.

To serialize an object, the object must implement the `Serializable` interface.

`Serializable` is a marker interface (an `empty` interface used to provide run-time information about objects), which is used to mark the class as serializable, so that the object of the class can be serialized and deserialized.

When object is serialized, the `serialVersionUID` will be generated automatically. The serialVersionUID is used to ensure that the serialized and deserialized objects are compatible. If the serialVersionUID is not the same, the deserialization will fail.

If we do not want to serialize some fields, we can use `transient` keyword to declare the field. Like `public transient int age`;

## Lambda expression

lambda expression is used to create an anonymous function.  
(para) -> {action}

The usage is in Java 8, we can use `lambda expression` to initialize a `functional interface`.

## Functional Interface

An interface that has only one abstract method and provides a single functionality. `@FunctionalInterface` annotation is not mandatory but it will add constraints to check if there is only one method in it.

- `Consumer` takes a single input and performs action without returning the result.

- `Supplier` used to generate a value.

- `Function` takes a single input and produces a result.

- `Predicate` takes a single input and returns a boolean value.

## Comparable & Comparator

They both are used for comparing two objects. They serve for different purposes.

- `Comparable` is used to sort objects in a natural order.

  1. Need to `implement` the `Comparable` interface and override the `compareTo()` method.

- `Comparator` is a functional interface used to sort objects in a custom order.
  1. We don't need to implement the `Comparator` interface, but we need to create a new class to implement the `Comparator` interface and override the `compare()` method.
  2. We can also use lambda expression to create a `Comparator` object.

## Singleton

There at most `one` instance exists during the application runtime. When there is a request, if the instance already exists, return the existing instance, else generate a new instance. Common example is the `bean scope` in Spring.

### How to create a Singleton

Use `static method` to create a singleton.

1. Eager initialization  
   Advantage : `Simple` and `thread safe`  
   Disadvantage: `Eager loaded` will cause memory waste, because the instance is created even if it is not used.

   ```Java
   public class Singleton {
       private static final Singleton instance = new Singleton();
       private Singleton() {}
       public static Singleton getInstance() {
           return instance;
       }
   }
   ```

2. Lazy initialization  
   Advantage : `Lazy loaded`, only create instance when it is needed, saves memory.
   ```Java
    public class Singleton {
         private static Singleton instance;
         private Singleton() {}
         public static Singleton getInstance() {
              if (instance == null) {
                instance = new Singleton();
              }
              return instance;
         }
    }
   ```

### Singleton with thread safe

Singleton is not thread safe, we need to use `synchronized` keyword to make it thread safe.

```Java
public class ThreadSafeSingleton {
    private static ThreadSafeSingleton instance;
    private ThreadSafeSingleton() {}
    public static ThreadSafeSingleton getInstance() {
        // Double-Checked Locking used to avoid obtaining the lock every time the code is executed
        if (instance == null) {
            synchronized (ThreadSafeSingleton.class) {
                if (instance == null) {
                    instance = new ThreadSafeSingleton();
                }
            }
        }
        return instance;
    }
```

## JDBC & statement

JDBC is a Java API used to connect and execute query to the database.
It uses JDBC statements to execute queries.

There are three types of JDBC statements:

- `Statement`: Used to execute static SQL statements with no parameters. Actually we can pass parameters by concatenating strings, but this is not recommended because it is vulnerable to SQL injection attacks. Such as `SELECT * FROM users WHERE username = 'admin' OR 1=1;` will return all the users. So we should use `PreparedStatement` instead.

- `PreparedStatement`: Used to execute static SQL statements with parameters. we can use `?` to represent the parameters, and then use `setXXX()` method to set the parameters. It can prevent SQL injection attacks.

- `CallableStatement`: Used to execute stored procedures in database.

## JDBCTemplate

`JDBCTemplate` is a class in Spring JDBC, which is used to simplify the use of JDBC and avoid repetitive code.

All we need to do is pass the `SQL statment`, `RowMapper` and `parameters` to the `JDBCTemplate` object, and then it will fetch the data from database and convert it to the object according to the rowmapper.

main methods:

- `queryForObject`: return a single object
- `query`: return a list of objects
- `update`: update the database

## Hibernate

Hibernate is an ORM (Object Relational Mapping) framework, which is used to map the object-oriented domain model to the relational database.

To configure Hibernate:

- Add the dependency in pom.xml

- Configure hibernate, we need a Hibernate `Configuration class`, inside it, we need to create a `sessionFactory` with the `dataSource`.

- Create entity class using `@Entity` and `@Table`, `@Id`, `@Column` `@GeneratedValue` to map objects to the database.

- To use it, we call sessionFactory.getCurrentSession to perform operations.

- To handle transactions, we can use @Transactional provided by Spring to handle transactions.

### Migration between different databases

Change database URL, driver, username, password, dialect

Dialect is used to generate the SQL statement according to the typr of database.

Driver is used to set up the connection between the application and the database.

### Entity State

- `transient` state, the entity instance is not attached to a session and has no corresponding rows in the database. It's usually just a new object that has been created to save to the database.

- `persistent` state, the entity instance is associated with a unique Session object (object already mapped to the database). Upon flushing the Session to the database, the entity is guaranteed to have a corresponding consistent record in the database.

- `detached` states, the entity instance was once attached to a Session, but now it’s not. An instance enters this state if it is evicted from the context, clear or close the Session, or put the instance through serialization/deserialization process.

### Fetching Strategies

- `Eager`: will fetch all attributes of an entity at once, including Aggregation / Composition.

- `Lazy`: **`Default`** will load the Aggregation / Composition fields only when requested. This can improve performance by reducing the amount of data retrieved from the database

`LazyInitializationException` occurs when accessing a lazily-loaded proxy outside the session. Hibernate is unable to fetch the data because the session has been closed. We can use eager fetching to prevent this from happening, or manually initialize the collection before the session closes.

### Hibernate Caching

There are two level Caching in Hibernate:

- `First level` cache provides caching at the session level. It is enabled by default. When you load an entity using a Session, Hibernate checks the first-level cache first to see if the entity is already loaded. If it's found in the cache, Hibernate returns the cached object, avoiding a database query. We can use session contains() to check if an obj is cached or not. get(). load() methods.

- `Second Level` cache is disabled by default, it is a higher level, cross-multi session. When query something, Hibernate will execute a query like select all fields from table where …, and cache all objects in second level cache, when hibernate need to query object by id, it will first check the first level cache, if not contains, then check the second level cache, if not contains too, it will fetch from database and put it into the second level cache. Second level cache is only used for querying by ID of Hibernate objects.

# Spring

## Spring Framework & Spring Boot

### Spring Framework

`Spring Framework`'s core feature is **`Dependency Injection (IOC)`**, **We don't need to initialize the object, spring framework will manage and inject the dependencies for us**. It also provides modules like `AOP`, `JDBC`, `ORM`, `MVC`, `Security` etc.

1. [Two types of IOC container](#two-types-of-ioc-containers):

   - `BeanFactory` : `lazy loaded`, `XML`
   - `ApplicationContext` : `eager loaded`, `XML & annotation`

2. [IOC container life cycle](#ioc-container-life-cycle):

   - When Spring application starts, IOC container is instantiated(BeanFactory or Application Context)
   - IOC container read the bean class from config files such as XML or annotations like @ComponentScan
   - Create bean instance based on the Bean definition
   - When a bean is required, inject the required dependencies.
   - Post-Post-initialization Pre-destruction and bean call back will be executed, then is the bean destruction when spring application shut down, then the container will shut down.

3. [Bean scopes](#bean-scopes):

   - `singleton` (default)
   - `prototype`
   - `request`
   - `session`
   - `global-session`

4. [Three types of dependency injection](#three-types-of-dependency-injection): (Optional)

### Spring Boot

`Spring Boot`'s core feature is **`Auto Configuration`**, which is used to reduce the time for setting up configurations.

1. talk about spring configuration is complex, spring boot enables auto configuration, which is to reduce the time for setting up configurations.

2. For the pom file, spring Boot will generate default dependencies for us like security, web, jdbc. We can also override the default dependencies by providing configuration files.

3. application.properties file, it is key value pairs, we can override the default configuration by providing the key value pairs in the application.properties file. like the data source url, username, password.

4. @SpringBootApplication annotation

   - `@EnableAutoConfiguration` : Allows auto configuration based on the .jar dependencies in the pom.xml file

   - `@ComponentScan` : scan the `@Component` and their child class like @Controller, @Repository under the basic folder.

   - `@Configuration` : Spring Boot will only scan @Bean third party class under the @Configuration class. This is used to add extra beans to the spring container.

## Spring MVC

1. When requst coming, DispatcherServlet map the request to the corresponding controller method.

2. In controller, service, dao, the data is processed recording to the business logic.

3. After that, the controller will create a model object contains the data in a key-value pair way. Controller will return a string of the view name to the view resolver.

4. View resolver has two responsibilities, first is to add the prefix and suffix to the view name to find the jsp page, second is to reach the template to replace the variables with the data in the model object. After that, the view resolver will return the view to the front end.

## Spring AOP

**AOP is used to separate functionalities that are not related to the core business logic.** In another word, it is used to **decouple cross-cutting concerns** from the core business logic. (e.g. logging, exception handling)

Spring AOP works by **using `proxy` to wrap the object being called.** The proxy is created by the Spring container and is used to intercept the calls to the object being proxied.

### 2.1. Join Point

`Join point` is a point in the execution of the application where an aspect can be plugged in. In another word, **it is the `unit of point` can be monitored.**

This point could be a `method being called`, an `exception being thrown`, or even a `field being modified`. In Spring AOP, a join point always represents a method execution.

### 2.2. Pointcut (Designator)

`Pointcut` is a predicate or expression that matches join points. It is a set of join points where `advice` should be executed.

### 2.3. Advice

`Advice` is the action taken by an aspect at a particular join point. Different types of advice include `before`, `after`, `after-returning`, `after-throwing`, and `around` advice.

### 2.4. Aspect

`Aspect` is a module that encapsulates join points, pointcuts, and advice. It is a class that implements cross-cutting concerns.

## Spring Security

`Spring Security` is a framework to protect our endpoints.

1. user want to login in with username and password: first it will be filtered by the filter chain.

- UserDetailsService loadUserByUsername() to load user information from database, and then compare the credentials.

- AuthenticationManager authenticate() to authenticate the user and generate UserDetails object with GrantedAuthority.

- JwtTokenUtil generateToken() to generate token with UserDetails.

2. user want to access the endpoint with token: first it will be filtered by the filter chain.

- JwtTokenUtil resolve() the token from request and generate the userDetails object.

- generate authentication object and set the authentication object to SecurityContext.

For authroization, we can use `@PreAuthorize("hasRole('ROLE_ADMIN')")` or `@PreAuthorize("hasAuthority('unverified')")` to check the role of the user. Also, we can use `@AuthenticationPrincipal UserDetails userDetails` to get the user details in the controller.

### JWT (JSON Web Token)

JWT is a standard that allows you to encode data in a JSON format. It is often used to encode authentication information, like the username, the permissions, etc. It is often used in combination with OAuth2.

JWT mainly contains three parts:

- Header: contains information about the type of token and the hashing algorithm used to sign the token.

- Payload: contains the claims, which are statements about an entity (typically, the user) and additional data. Claims can be things like the username, the permissions, etc.

- Signature: is used to verify that the sender of the JWT is who it says it is and to ensure that the message wasn’t changed along the way. (Created by hashing the header and the payload with a secret key)

## Mockito

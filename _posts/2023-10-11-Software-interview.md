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

## OOP

### Encapsulation

**Definition** : Encapsulation is used to hide the internal details of a class. Only provide essential information to the users and hide implementation details.

Java implements it by making the fields in a class private and providing access to the fields via public methods like reflection. This helps to maintain data integrity and prevent unauthorized access to or modification of an object's internal state. Improve security, easier maintenance, improve code reuse.

E.g. `private` fields and `public` getters and setters.

### Inheritance

**Definition** : Derive something specific from something generic, a class can inherit all properties and methods of another class and add its own modification.

Use `Extend` keywords to declare inheritance, inheritance in Java implies that there is an IS-A relationship between the classes.

Type: single, multiple, multilevel, hierarchical, hybrid. Java doesn’t support Multiple inheritance

#### Diamond Problem

Multiple inheritance will cause diamond problem
The diamond problem in Java is about multiple inheritance. `When two superclasses of class A have a common parent class`, they all override the method of the parent class. When class A calls that method, it is ambiguous, so here arises a problem.

1. Interface: We can use the default methods and interfaces to achieve multiple inheritance, since Java supports more than one class implementing the interface.

2. aggregation: We can also use aggregation to include objects of multiple classes within its own implementation. This allows the class to access the methods and properties of the aggregated objects without directly inheriting from them, thus avoiding the conflicts and ambiguities of multiple inheritance.

### Polymorphism

**Definition** : we can perform a single action in different ways
Java implements it by `overriding` and `overloading`.

#### method overriding & method overloading Provide your own example

1. Overriding is to redefine a method that has been defined in a parent class, with the same signature.

2. Overloading is to define two methods with the same name, in the same class, but with different signatures.

### Abstraction

**Definition** : Abstraction is used to hide the internal details and show only the functionality to the users.

#### interface and abstract class

1. Interface is a blueprint of a class that have static constants and abstract methods. It can only have abstract methods, and all methods are public and abstract by default. It is used to achieve abstraction and multiple inheritance.

2. Abstract class is a class that is declared abstract, it can have abstract and non-abstract methods. It is used to achieve abstraction and code reusability.

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

### SynchronizedMap & ConcurrentHashMap & HashMap

<img src = "/images/Full-Stack/JavaCore/HashMap.png">

`HashMap`: not Thread-safe, can have one null key and multiple null values.  
`SynchronizedMap`: Thread-safe, Slow Performance, can have one null key and multiple null values. Can only have one thread reading.  
`ConcurrentHashMap`: Thread-safe, Fast Performance, null key and values are not allowed. Can have multiple threads reading at the same time, because it has multiple segments.

## Exception

### Checked Exception & Unchecked Exception

There are two types of exceptions in Java, checked exceptions and unchecked exceptions.

- `Checked exceptions` are checked at compile-time. It means if a method is throwing a checked exception then it should handle the exception using try-catch block or it should declare the exception using throws keyword, otherwise the program will give a compilation error.  
  Example: `IOException`, `SQLException` etc.

- `Unchecked exceptions` are not checked at compile-time, it means if your program is throwing an unchecked exception and even if you didn’t handle/declare that exception, the program won’t give a compilation error. It is up to the programmer to judge the conditions in advance, that can cause such exceptions and handle them appropriately.  
  Example: `NullPointerException` etc.

### Handle Exception

- For handle exception, we can use `try-catch` block to handle the exception, or use `throws` keyword to propagate the exception to the caller.
- In Spring Boot, we can use `@ExceptionHandler` to handle the exception.

### Exception Propagation

When an exception happens (like throw exceptions or runtime error), first it will check in current functions, is there any handler like try catch block to handle the exception, if not handled, it will be thrown to the caller function in the call stack, again and again until it is handled.

### Custom Exception

Create a class extending `Exception` or `RunTimeException` class, to use it, throw an exception in somewhere and use a try catch block to catch it.

```Java
public class CustomException extends Exception {
   public CustomException(String message) {
      super(message);
   }
}
```

## Thread

### LifeCycle of Thread and common knowledge

<img src="/images/Full-Stack/JavaCore/ThreadState.png">

1. `New`: When we create a new thread, it is in the `new` state.

2. `Runnable`: When we call the `start()` method, the thread will be in the `runnable` state. In this state, the thread is ready to run, but it is not running yet.

3. `Running`: When the thread is running, it is in the `running` state.

4. `Blocked`: The thread will be in blocked state when it is trying to acquire a lock but currently the lock is acquired by the other thread. The thread will move from the blocked state to runnable state when it acquires the lock.

5. `Waiting`: When we call the `wait()`, `join()` method, the thread will be in the waiting state and `release the lock` until another thread calls the `notify()` or `notifyAll()` method.

6. `Timed Waiting`: When we call the `sleep()` method, the thread will be in the timed waiting state and `release the lock` for the specified time. When the time is up, the thread will move to the runnable state.

7. `Terminated`: When the thread finishes its execution, it will be in the terminated state.

#### why wait() and notify() are in Object class but not in Thread Class?

Because they are used to manage synchronization and communication between threads that are interacting with the same object.

The purpose of wait() is to make a thread temporarily release the lock on the object it is associated with, allowing other threads to enter synchronized blocks or methods associated with the same object. notify() is used to wake up one waiting thread that is waiting on the same object, giving it a chance to acquire the lock and proceed.

If wait() and notify() were part of the Thread class, it might lead to confusion and difficulties when trying to coordinate threads on specific objects.

#### Join() and Sleep()

1. join() is used to let the current thread wait for others to finish.

   When a thread invokes join(), it will block itself until the target thread completes execution. This helps you make sure all other threads finish their work and then proceed to execute the current thread.

2. sleep() is used to let the current thread sleep for a specified time.

   When a thread invokes sleep(), it will block itself for the specified time. This helps you pause the execution of the current thread for a specified time.

#### Deadlock

Deadlock is a situation where two or more threads are blocked forever, waiting for each other to release the lock. Use a join() method between two start() to resolve this situation, so that the main will wait for the first thread to finish and then start the second thread. Use synchronize.

Strictly deadlock has 4 necessary conditions

1. Mutual Exclusion: A resource can be held by only one process at a time.
2. Hold and Wait: A process can hold a number of resources at a time and at the same time, it can request for other resources that are being held by some other process.
3. No preemption: A resource can't be preempted from the process by another process, forcefully.
4. Circular Wait: Circular wait is a condition when the first process is waiting for the resource held by the second process, the second process is waiting for the resource held by the third process, and so on.

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

# Java 8

Name some new features in Java 8:

- Lambda expression

- Functional interface

- Stream API

- Optional

- Default and static methods in interface

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

## Stream API

Stream API is used to process collections of objects, it is a pipeline that performs a series of operations to the object to get the desired result.

### Terminal Operations & Intermediate Operations (API)

Intermediate operation is lazily executed and returns a stream as a result, thus can be pipelined. `filter()` , `map()` , `flatMap()` , `distinct()` , and `sorted()`.

flatMap() : It can transform data or flatten data, like flatten a 2d array to an array.

<img src= "/images/Full-Stack/JavaCore/StreamAPIIntermediate.png">

Terminal operation is eagerly executed and returns a non-stream result, thus ending the stream. `forEach()` , `count()` , `collect()` , `reduce()` , `allMatch()` , `anyMatch()` , `noneMatch()` , `findFirst()` , `findAny()`.

<img src= "/images/Full-Stack/JavaCore/StreamAPITerminal.png">

### Parallel Stream

By default, stream api is sequential, but we can set it to parallel.

Instead of using .stream(), use parallelStream() to create a parallel stream. Other operations are similar to the sequential stream. operations may be executed concurrently by multiple threads, which can lead to faster processing for certain tasks

# Design Pattern

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

````

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

`Mockito` is a `mocking framework` used for `unit testing` of Java applications. When we write unit tests, we need to mock the `external dependencies`. Mockito provide an easy way to mock the dependencies.

### Difference between `@Mock`, `@Spy`, `@InjectMocks`

`@Mock` is used to create and inject mocked instances. Often used to mock the dao layer. It will not call the real method inside.

`@InjectMocks` will inject the mocked instances into the tested object automatically. Often used to mock the service layer. It will call the real method inside. (suppose we are testing service layer, the service should be marked as `@InjectMocks`. becasue we need to run the logic inside each method)

`@Spy` is used to create and inject partially mocked instances. It will call the real method inside. It is used when we want to call the real method and perform some logic inside the method. like sending an email and then test if the email is sent successfully.

### What is the difference between `@Mock` and `@MockBean`? (same with `@Spy` and `@SpyBean`)

They both are used to create a fake object. `@Mock` create object not related to the spring framework.

When testing controller, it need us to start the Spring Boot application. We use `@MockBean` to create a fake object that is part of the Spring context. such as beans declared in the application's configuration or beans created by Spring Boot auto-configuration.

## Monolithic vs Microservices

Which one to use depends on the business requirements and the size of the application.

### Monolith

Monolithic means asingle server handle multiple business logic. It is tightly coupled. It is suitable for small applications that do not need to scale or handle high throughput. It is easy to develop and deploy compared to microservices.

### Microservices

Microservice is a distributed architecture, each service responsible for a single business logic. It is loosely coupled which is easy to add more features and scale specific components independently. The disadvantage is that it needs well designed and more complex to develop.

## Communication between Microservices

### Synchronous Communication

#### HTTP Requests

Microservices communicate with each other using HTTP requests. Each microservice has its own REST API. In Spring Boot, we can use the `RestTemplate` class to make HTTP requests.

#### Eureka and Feign

`Eureka` is a service registration tool. It allows microservices to register themselves and discover other services. Each service send a heartbeat to Eureka to let it know that it is still alive. If Eureka does not receive a heartbeat from a service, it will remove it from its registry.

`Feign` is a REST client to make HTTP requests to other microservices without using `RestTemplate`. It uses Eureka to discover other services. In this way, we can make HTTP requests to other services without hardcoding the URL, also simplifying the code.

### Asynchronous Communication

#### Message Brokers

We can use a `message broker` like `RabbitMQ` or `Kafka` to send messages between microservices to communicate `Asychonously`.

For example, when a user creates a new account, we can send a message to the email service to send a welcome email and to the notification service to send a notification to the user
````

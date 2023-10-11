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

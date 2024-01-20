---
layout: post
title: "Java Basic"
date: 2023-01-20
description: "Java Basic knowledge"
tag: Interviews
---

## SOLID Principles

[detailed explanation](https://chriszzhong.github.io/2023/03/Java-Core/#solid-principle)

- `Single Responsibility Principle`: A class should have only one job.

- `Open Closed Principle`: Class open for extension, closed for modification.

- `Liskov Substitution Principle`: Subclass must be substitutable for their parent class.

- `Interface Segregation Principle`: If an implementation class does not use all the methods of an interface, then the interface should be split into smaller ones.

- `Dependency Inversion Principle`: High-level modules should not depend on low-level modules. Both should depend on abstractions.

## OOP

[detailed explanation](https://chriszzhong.github.io/2023/03/Java-Core/#oop)

- `Encapsulation`: wrapping up of data under a single unit. (data hiding, `reflection`)

- `Abstraction`: hiding internal details and showing functionality only. (`abstract class`, `interface`)

- `Inheritance`: Class can extend another class. (`Diamond problem`)

- `Polymorphism`: perform a single action in different ways. (`overloading`, `overriding`)

## Data Structure

### Array & ArrayList

| -            | Array                                      | ArrayList                             |
| ------------ | ------------------------------------------ | ------------------------------------- |
| Type         | Array                                      | implemet List                         |
| Size         | fixed size                                 | dynamic size                          |
| Element type | can store primitive data types and objects | can only store objects                |
| Performance  | Slightly better than ArrayList             | slower, More flexible and ease of use |

### ArrayList & LinkedList

| -           | ArrayList                                   | LinkedList                                |
| ----------- | ------------------------------------------- | ----------------------------------------- |
| Performance | faster, random access O(1) get(index)       | slower, sequential access                 |
| Usage       | frequently need to access elements by index | Frequently insert/remove at head or tail. |

### HashMap

- HashMap maintains a bucket array to store the key-value pairs.
- `Hashing`: when adding a key-value pair, it generates a hash code according to the key; the code is used to determine the index of the bucket.
- `Index calculation`: taking the modulus of the hash code with the size of the bucket array.
- `Collision`: when two keys have the same hash code, they are stored in the same bucket as a linked list.

### HashTable & SynchronizedMap & ConcurrentHashMap

- `HashMap`: not thread-safe, fast, null key/value.
- `SynchronizedMap`: thread-safe, slow, null key/value.
- `HashTable`: thread-safe, slow, no null key/value.
- `ConcurrentHashMap`: thread-safe, fast, No null key/value.

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

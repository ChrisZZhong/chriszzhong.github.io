---
layout: post
title: "Transactions in Spring (monolithic)"
date: 2023-04-08
description: "Transactions"
tag: Spring Framework
---

# Transactions in Spring (monolithic)

## 1. Transaction

Transaction represents a single unit of work. The ACID properties define transactional requirements.

- **`Atomicity`**: All operations within the transaction must succeed or fail as a unit. If failed, the transaction must be rolled back to the state before the transaction was started.

- **`Consistency`**: Database must be consistent before and after the transaction.  
  EG : Suppose A and B have $1000 each. If A transfers $500 to B, no matter the transaction succeeds or fails, the total amount of money in the bank must be $2000.

- **`Isolation`**: Multiple transactions can occur concurrently without leading to inconsistency.  
  EG : Suppose A and B have $1000 each. If A transfers $500 to B, and at the same time, B transfers $500 to A, the two transactions must be executed independently. The final amount of money in the bank must be $2000.

- **`Durability`**: After the transaction has completed, its effects are permanently in place in the system. The modifications persist even in the event of a system failure.

## 2. Spring Boot Transaction

### 2.1. @Transactional

`@Transactional` is used to annotate a method as a transactional method. It can be used on both class and method level. If used at class level, all the public methods of the class are transactional. If used at method level, only the annotated method is transactional.

Also, we need to configure `@EnableTransactionManagement` at the entry point of the application.

For transactional in distributed system, see here: [Distributed Transactions](https://chriszzhong.github.io/2023/06/transaction-in-microservices/)

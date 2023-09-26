---
layout: post
title: "Srping AOP"
date: 2023-04-10
description: "Srping AOP"
tag: Spring Framework
---

# AOP - Aspect Oriented Programming

## 1. What is AOP?

**AOP is used to separate functionalities that are not related to the core business logic.** In another word, it is used to **decouple cross-cutting concerns** from the core business logic.

For example, logging is a cross-cutting concern because it is used in many places in the application. It is not related to the core business logic. So, it is a good idea to separate logging from the core business logic. This is where AOP comes into the picture.

## 2. AOP Concepts

### 2.1. Join Point

`Join point` is a point in the execution of the application where an aspect can be plugged in. In another word, **it is the `unit of point` can be monitored.**

This point could be a `method being called`, an `exception being thrown`, or even a `field being modified`. In Spring AOP, a join point always represents a method execution.

### 2.2. Pointcut (Designator)

`Pointcut` is a predicate or expression that matches join points. It is a set of join points where `advice` should be executed.

### 2.3. Advice

`Advice` is the action taken by an aspect at a particular join point. Different types of advice include `before`, `after`, `after-returning`, `after-throwing`, and `around` advice.

### 2.4. Aspect

`Aspect` is a module that encapsulates join points, pointcuts, and advice. It is a class that implements cross-cutting concerns.

## Spring AOP works

Spring AOP works by **using `proxy` to wrap the object being called.** The proxy is created by the Spring container and is used to intercept the calls to the object being proxied. This is a `decoupling` work to monitor the application **without modifying the core business logic**.

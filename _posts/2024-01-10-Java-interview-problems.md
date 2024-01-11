---
layout: post
title: "Java interview questions"
date: 2023-12-28
description: "frequent questions asked in interviews"
tag: Interviews
---

## Spring Framework

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

Bean is object managed by Spring IOC container.

- `Singleton` : `default scope` only created once during Spring application run time
- `Prototype` : new instance per request
- `Request` : new instance per HTTP request
- `Session` : new instance per HTTP session
- `GlobalSession` : new instance per global HTTP session

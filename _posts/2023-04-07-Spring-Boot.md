---
layout: post
title: "Spring Boot"
date: 2023-04-07
description: "Spring Boot"
tag: Spring Framework
---

# Spring Boot (knowledge)

## Spring Boot VS Spring Framework **(interview question)**

**In interview:** Answer the `Feature` of Spring Boot and Spring Framework:

1. `Spring Framework`'s core feature is **`Dependency Injection (IOC)`**, **We don't need to initialize the object, spring framework will manage and inject the dependencies for us**. It also provides modules like `AOP`, `JDBC`, `ORM`, `MVC`, `Security` etc.

   Talk more about the Dependency Injection **[Here](https://chriszzhong.github.io/2023/04/Dependency-Injection/)**

2. `Spring Boot`'s core feature is **`Auto Configuration`**, which is used to reduce the time for setting up configurations.

   Talk more about the Auto Configuration **[Here](#auto-configuration-springbootapplication)**

## Auto Configuration in Spring Boot **(interview question)**

1. talk about spring configuration is complex, spring boot enables auto configuration, which is to reduce the time for setting up configurations.

2. For the pom file, spring Boot will generate default dependencies for us like security, web, jdbc. We can also override the default dependencies by providing configuration files.

3. application.properties file, it is key value pairs, we can override the default configuration by providing the key value pairs in the application.properties file. like the data source url, username, password.

4. @SpringBootApplication annotation

   - `@EnableAutoConfiguration` : Allows auto configuration based on the .jar dependencies in the pom.xml file

   - `@ComponentScan` : scan the `@Component` and their child class like @Controller, @Repository under the basic folder.

   - `@Configuration` : Spring Boot will only scan @Bean third party class under the @Configuration class. This is used to add extra beans to the spring container.

## why Spring Boot?

Spring framework provides a comprehensive programming and configuration model for Java based applications, It integrates well with a lot of components like IOC, AOP, JDBC, ORM, security.

MVC architecture, which is model view and controller is for web applications. It has predefined templates to connect to the database, it achieves loose coupling through the dependency injection.

One of the cons is that the configuration of the spring framework is very complex, you have to write a huge amount of XML files which greatly slows down development. For example, you should manually put the bean objects in the approties.xml file, for Mybatis, you should manually write sql and map the result to the pojo class.To solve this problem, the sun company, the owner of the spring, gave a solution, Spring Boot.

Spring Boot is a framework that is built on top of Spring Framework, it uses starter dependencies and enables annotation based configuration (that is auto config) which greatly reduces the time for setting up configurations.

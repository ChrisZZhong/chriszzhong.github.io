---
layout: post
title: "Switching Configurations and Jasypt"
date: 2023-04-16
description: "Switching Configurations and Jasypt"
tag: Spring Framework
---

# What annotation do you use to quickly switch between different environments to load different configurations?

`@Profile` is used to quickly switch between different environments to load different configurations.

- Add `@Profile` annotation to the configuration class.

  ```java
    @Configuration
    @Profile("dev")
    public class DevConfig {
        // ...
        @Value("${spring.datasource.url}")
        private String url;
    }
  ```

- Add `spring.profiles.active` in `application.properties` file.

  ```properties
    spring.profiles.active=dev
  ```

# What is Jasypt?

`Jasypt` is a java library which allows the developer to perform encryption and decryption in Java application. Such as password, database connection strings, etc.

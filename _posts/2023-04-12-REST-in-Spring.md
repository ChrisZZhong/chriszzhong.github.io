---
layout: post
title: "REST Controller in Spring"
date: 2023-04-12
description: "REST Controller in Spring"
tag: Spring Framework
---

# REST Controller in Spring

## @RestController VS @Controller

`@RestController` is a convenience annotation that combines `@Controller` and `@ResponseBody`. It is used for building RESTful web services that return data in response to HTTP requests in JSON/XML. We don't need to add `@ResponseBody` to our request mapping methods one by one.

`@Controller` is a common annotation that is used to mark a class as Spring MVC Controller while `@ResponseBody` is used to indicate that the return value of a method should be used as the response body of the request.

## @RequestBody VS @ResponseBody

`@RequestBody` is used to map the content of an HTTP request body to a Java object, so you can access the data sent by the request in Java.

`@ResponseBody` is used to map the return value of a method to an HTTP response body, so you can return Java objects as data in the response.

## RestTemplate

`RestTemplate` is a tool provided by Spring to simplify the HTTP request to external Restful API.

The advantage is that it is easy to use, but the disadvantage is that the request URL is hardcoded, not good for decoupling. Also, it is synchronous, which means that the thread will be blocked until the request is finished.

## How to validate the values of the request body? How does BindingResult work?

1. For the DTO classes, we can use `@NotNull`, `@NotEmpty`, `@Size`, etc. to validate the values of the fields.

2. In the controller method, we can use `@Valid` to validate the values of the request body.

`BindingResult` is used to validate the values of the request body. It is used to store the result of the validation and has to be placed right after the request body parameter. We can use `hasErrors()` to check if there is any error and handle the error in the way we want.

# REST Principles. See [here](https://chriszzhong.github.io/2023/04/RESTFulAPI/)

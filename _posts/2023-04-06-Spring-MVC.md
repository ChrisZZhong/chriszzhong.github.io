---
layout: post
title: "Spring MVC"
date: 2023-04-06
description: "Spring MVC"
tag: Spring Framework
---

# Spring MVC (All important points)

## What is Spring MVC?

Spring MVC is a framework built on top of the Spring framework, it is a web framework that is used to build web applications. It is based on the MVC architecture, which is `model` `view` and `controller`.

- `Model` is the data layer, it is used to store the data, it is usually a POJO class.

- `View` is the presentation layer, it is used to display the data, it is usually a JSP page.

- `Controller` is the business layer, it is used to process the data, it is usually a servlet.

## Workflow of Spring MVC (Dispatcher servlet)

1. When requst coming, DispatcherServlet map the request to the corresponding controller method.

2. In controller, service, dao, the data is processed recording to the business logic.

3. After that, the controller will create a model object contains the data in a key-value pair way. Controller will return a string of the view name to the view resolver.

4. View resolver has two responsibilities, first is to add the prefix and suffix to the view name to find the jsp page, second is to reach the template to replace the variables with the data in the model object. After that, the view resolver will return the view to the front end.

## View Resolver

View resolver has two responsibilities:

1. First is to add the prefix and suffix to the view name to find the jsp page

2. Second is to reach the template to replace the variables with the data in the model object.

After that, the view resolver will return the view to the front end.

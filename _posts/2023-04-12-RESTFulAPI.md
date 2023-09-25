---
layout: post
title: "RESTFul API"
date: 2023-04-12
description: "RESTFul API"
tag: Computer Network
---

# RESTFul API

## What is RESTFul API (Application Programming Interface)?

You can think of an API as a mediator between the users or clients and the resources or web services they want to get.

### What is REST?

REST is an architectural style that defines a set of rules in order to create Web Services for communication between different servers.

### RESTFul API?

RESTFul API is an API that follows the REST architectural style.

## RESTFul API 6 principles

1. `Uniform Interface` : All API requests for the same resource should look the same, no matter where the request comes from.

2. `Client-Server decoupling` : client and server applications must be completely `independent` of each other. Client is responsible for the `user interface` and server is responsible for the `backend data storage`. They communicate through a REST API via HTTP requests.

3. `Stateless` : the server does not store any client context data. Each request from the client to the server must contain all the information necessary to understand the request. The server cannot take advantage of any previous requests sent by the client.

4. `Cacheability` : When possible, resources should be cacheable on the client or server side. Server responses also need to contain information about whether caching is allowed for the delivered resource. The goal is to improve performance on the client side, while increasing scalability on the server side.

5. `Layered System` : The client cannot tell whether it is connected directly to the server or to an intermediary along the way. This means that the client does not need to know the internal structure of the system.

6. `Code on demand` : The server can provide executable code or scripts for the client to execute in its context. This constraint is the only one that is optional.

# REST VS SOAP

## SOAP

SOAP is a protocol that uses XML to exchange information between computers over the Internet. It is an XML-based protocol for accessing web services. Compared to REST, SOAP is more difficult to implement and requires more bandwidth and resources.

Soap document need to have a root element called `<Envelope>`, which is required, and `<Header>` and `<Body>`, which are optional.

1. Header: contains routing and processing information.
2. Body: contains the actual message.

# REST Controller in Spring. See [here](https://chriszzhong.github.io/2023/04/REST-in-Spring/)

---
layout: post
title: "mphasis interview questions"
date: 2023-12-28
description: "frequent questions asked in interviews"
tag: Interviews
---

## what is message queues

message queues can be used for `asynchronous communication` between microservices. it is used to `store messages` until the consumer is ready to consume the message.

message queues `decouple` the services and improve the `performance`. Endpoints producer and consumer interact with the queue, not each other. Producers can `add requests` to the queue without waiting for them to be processed. Consumers process messages only when they are available. No component in the system is ever stalled waiting for another, optimizing `data flow`.

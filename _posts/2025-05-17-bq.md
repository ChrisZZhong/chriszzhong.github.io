---
layout: post
title: "intro"
date: 2022-05-22
description: "intro"
tag: Interviews
---

## Introduction

I'm Chris, I have 2 year experience as software developer.
I have experience in building microservice applications with Java, JavaEE, Spring Boot or Spring MVC, and also have experience with gRPC.
For testing tools, I've used Junit, Mockito, postman regression scripts.
For databases, MySQL, Oracle DB, SQL Server, RDS, protgres
For cloud, AWS S3 EC2, RDS

now I'm working for madison-davis, our client is bank of china, The team was tring to migrate legacy SOAP services to RESTful services. I was assigned an e-commerce web service, handling the credit card transactions for the child banks and credit unions.

I'm also an individual developer running a website for the software tech skills, the stack is next.js and springboot.

-----------------

legacy services are all on linux and no centralized log, The legacy server in production is started at 2008, and there is no centralized log. so whenever there is an incident, we are not able to localte it quickly, the new restful service integrate splunk with tracing ID, it is easy to tracking the status between gateway, backend and MQ layers.

We have restful core services called T24 and other services are built on top of that and use Api and MQ to interact with them to manipulate data.

## Difficult project

In my recent project, I am responsible for refactoring an e-commerce backend service from SOAP to REST. One key feature was sending transactional emails to users based on the API input type—for example, sending a receipt email after a successful payment.

However, during peak hours, we started receiving incident reports where some users weren't getting their receipt emails, and others experienced significantly delayed page responses after payment. This created a poor user experience and impacted user trust.

I began by reviewing the logs and analyzing the legacy code. I discovered that the email functionality was directly embedded in the payment service and used SMTP for sending emails. Because the email logic was tightly coupled and synchronous, the system would wait for the email to be sent before completing the payment flow. Under high traffic, this became a bottleneck. Additionally, the email service couldn't scale independently, leading to overload and failures.

To address this, my first step was to decouple the email functionality from the payment flow. I proposed introducing a message queue between the payment service and the email notification system. This would allow payment processing to complete quickly, while email sending could be handled asynchronously.

Before implementing this, I consulted with my team lead to check if we had any existing infrastructure we could leverage. Fortunately, we already had a centralized notification service in production that supported scalable email delivery. This meant we didn’t need to build and maintain a new service from scratch.

I collaborated with teammates to gather feedback and ensure alignment. To validate the design, I built a simplified prototype that demonstrated the asynchronous flow using the message queue and notification service.

After we deployed the new flow, we saw a significant reduction in email-related incidents, and user-facing latency during payment dropped noticeably. This not only improved the system's reliability and scalability but also enhanced the overall user experience.

## CI/CD workflow

For local, write code, pass the code style check, meet 100% code coverage, push to your branch, trigger the build and verify on Jenkins.

Once merged to master, will trigger the following:

1. Build and Verify: code analysis, execute unit tests, build project artifact.
2. Scan: security scan on code using Fority, the vuneralbilities are scanned here.
3. Release: release the artifact to the Sonatype Nexus maven central
4. Deploy: auto-deployed and configured to the DEMO environment using rundeck job
5. Regression: start regression tests on DEMO
6. QA: deployed to QA and do the QA regression

## Difficult team members (conflict / diff opinion)

I haven’t experienced working with difficult team members but sometimes we hold different opinions about things. In my recent project at Fiserv, there was a time I had a difference of opinion with one of my colleagues over the choice of a message broker for the provider.

Initially, I proposed using AWS (SQS) because it seemed like a convenient option given our existing infrastructure on AWS. However, my colleague suggested using Kafka instead.

To address this, I scheduled a one-on-one meeting with him to discuss our viewpoints. During the meeting, I explained why I preferred AWS SQS, emphasizing its compatibility with our current cloud services, like RDS and S3. I argued that even using kafka, we still need to deploy it somewhere. My colleague, on the other hand, was concerned about the potential risk of `vendor lock-in`. He made a valid point about the potential risks of relying solely on AWS, especially if some day aws goes down. He believed that using Kafka offered more flexibility for future migrations, such as using GCP.

Considering things in the long-term such as potential future migrations to GCP, became a key takeaway from this experience. I also learned to make decisions not only based on project requirements but also with a forward-thinking mindset.

Instead of being stubborn about my viewpoint, I chose to use kafka which efficiently handled the messages and enhanced the overall performance of our application.

## Deadline Problem (Mistake, slove, learned) (vulnerability)

This thing rarely happens to me. But when I am at Fiserv, there was a time I almost missed the deadline, I had a ticket to fix a high vulnerability, and while I was in the middle of debugging, I met a roadblock, something wrong with the Api. Fortunately we have been paying close attention to the logs, this helps a lot to narrow down the scope. To figure out what happened, I debugged the logs of the service carefully to make sure other functions in services worked fine and finally found that the the api the server called does not work well as expected. At that point, I realized that the downtime could significantly delay my progress.

So, I reported the situation to my manager. At the same time, I reached out to the coworker responsible for that service. I explained the urgency of the situation, and we set up a meeting and estimated the recovery time together. In the end, my manager rescheduled the deadline for my ticket.

Luckily, the server recovered the next day, there was no significant impact on business, and I was able to complete the ticket on time.

In this incident, What I took away was the importance of good communication. Keeping everyone in the loop, collaborating with my teammate to estimate the recovery time and discuss the solutions also contributes a lot.
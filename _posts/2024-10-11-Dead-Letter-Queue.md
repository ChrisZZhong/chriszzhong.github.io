---
layout: post
title: Dead Letter Queue(DLQ) Handler Service
date: 2024-05-21
description: "Dead Letter Queue Handler"
tag: Distributed system
---

## Dead Letter Queue

A Dead Letter Queue (DLQ) is a service implementation utilised in message-based systems to store messages that could not be processed or delivered.

For consumer, there may be scenarios in which incoming messages cannot be fully processed due to corruption in the incoming event, faults in the application code, temporary resource outages, etc. **Since Kafka events are removed from the topic when consumed, we need a way to prevent permanent loss of the event data in the scenario it cannot be processed.**

This microservice serves as a generic error handling solution across Kafka microservices that accept forwarded errored events and persist them for later reprocessing once the original issue has been rectified.

### Implementation Overview:

This microservice is built with Spring Boot and integrates a Kafka consumer and producer to facilitate message flow to and from a configured consumer application.

Incoming errored events are ingested from a common DLQ topic, written to the DLQ, and forwarded to a microservice-specific retry topic once ready to be reprocessed. Client microservices are configured to use the DLQ flow by supplying a mapping from their origin topic to their retry topic and providing their associated Avro schemas for event reconstruction.

The DLQ persistence layer is implemented in a database table BW_MS_RTTP.MS_EVENT_ERROR_LOG, storing all requisite information need to reprocess the event at a later stage.

<img src="/images/Kafka-DLQ/DLQ-Kafka.png">
<!-- ![image](https://github.com/user-attachments/assets/96c031b3-ca6e-46cc-8bc0-056db1e97ef3) -->

### Data Flow

- The microservice is composed of two concurrent flows:
  - Consumer flow: ingests events from the DLQ topic and persists them to DLQ table along with all necessary metadata required to route the original event for reprocessing.
  - Producer flow: polls for events which are ready to be reprocessed and publishes event to appropriate retry topic

**Activity Diagram:**
<img src="/images/Kafka-DLQ/activity-diagram.png">

<!-- ![image](https://github.com/user-attachments/assets/cb2b347d-745f-42dc-9e44-2b551641335b) -->

**Consumer-Side Flow:**

- **Data Ingested From**: DLQ topic
- **Processing Steps**:
  - The entry-point of this flow is DeadLetterQueueServiceConsumer.java
  - A **@KafkaListener** reads events from the configured DLQ topics and passes them to the **DeadLetterQueueService**.
  - The event is mapped to a DLQ entry entity and persisted.
    - Note that the body of the event is serialised as JSON and not Avro.
- **Data Outputted To: BW_MS_RTTP.MS_EVENT_ERROR_LOG** (DLQ table)

**Producer-Side Flow:**

- **Data Ingested From: BW_MS_RTTP.MS_EVENT_ERROR_LOG** (DLQ table)
- **Processing Steps**:
  - The entry-point of this flow is **DaemonConfiguration.java**.
  - This class instantiates a poller to fetch records with **STATUS = 'RETRY'** and **RETRY_ATTEMPTS <= 3** from the DLQ table.
  - The events are passed to **EventRetryHandler.processEvent()**, where the event schema is read and the corresponding Java generated model is determined by reflection.
    - This requires that the original consumer service's Avro schema is available on the classpath (e.g. in src/main/resources).
  - An instance of the Java Avro model for the schema is populated from the JSON body via a **jsonAvroBidirectionalMapper**.
  - The event's retry topic is pulled from config using the source topic and the event data is used to construct a Kafka **GenericRecord**.
  - The **RetryEventPublisher.java** implementation then handles publishing to the retry topic.
    - On success, we mark the event as **STATUS = 'PROCESSED'**
    - On failure, we increment the **RETRY_COUNT**
- **Data Outputted To**: A Kafka retry topic, determined by the event source.

**Retry Criteria**

- Events must have **STATUS = 'RETRY'** to be considered for reprocessing.
- Events must have **RETRY_ATTEMPTS < 3**

**example headers**

| header Name                      | Description                                                                                                                                                                                                                | Required                                                    |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| dead-letter-exception-class-name | Exception class name from where the event failed.This will be use to classify the exceptions as retriable or non-retriable.eg.TransactionDataProcessingException/DatabaseProcessingException/KafkaDeserializationException | YES                                                         |
| dead-letter-offset               | Event offset number for the event for the original topic. This could either be from primary topic or the retry topic                                                                                                       | YES                                                         |
| dead-letter-partition            | Topic partition number for the event for the original topic. This could either be from primary topic or the retry topic                                                                                                    | YES                                                         |
| dead-letter-reason               | Event failure reason. eg: com.fiserv.omnipay.exception.TransactionDataProcessingException: Processing of the transaction failed at transactionDataService: [Attribute:fdms80_0607_4_31].                                   | Optional Incase of deserialization issue this could be null |
| dead-letter-reason               | Additional information if present in addition to failure cause                                                                                                                                                             | Opt                                                         |
| dead-letter-schema-name          | Original Topic Avro Schema name eg. TransactionCanonicalModel                                                                                                                                                              | Opt                                                         |
| dead-letter-schema-version       | Schema version eg: 2.1.0                                                                                                                                                                                                   | Opt                                                         |
| dead-letter-source               | Name of the service which published the dlq events eg omnipay-fee-consumer-service                                                                                                                                         | YES                                                         |
| dead-letter-topic                | Original Topic name from where the event was consumed                                                                                                                                                                      | YES                                                         |

### blogs

[DLQ in Kafka](https://www.kai-waehner.de/blog/2022/05/30/error-handling-via-dead-letter-queue-in-apache-kafka/)

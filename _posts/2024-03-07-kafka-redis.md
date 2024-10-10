---
layout: post
title: "kafka & redis"
date: 2024-03-07
description: "kafka & redis"
tag: distributed system
---

## Kafka & Redis cache

![f334eb61dc310e91d74a5f8d0ddf224](https://github.com/user-attachments/assets/ae543270-2f19-4435-97e6-251a28313228)

### Kafka Workflow

(Two type of events: bulk load & change data)

```
                  DB2 mainframe
                          | retrieve data
            bulk load     |                                                                       save to redis
Client Api ---------->  kafka ---------------> Kafka cluster topic <------------------ Consumer -------------> Redis
                                produce message                       comsume topic message

Event types:
Bulk load,
Change data
Delete data
```

Client call api to trigger Bulk load, Kafka process an incoming request, build a record (client account setting, card ) and publish it to Kafka topic(voltage encrypted) (read from ODS, producer process client ODS messages send to topic), generate an unique key for client to retrieve data from Redis.

Use @Schedule for consumer, consumer subscribe topic, send every 100 count record to Redis, and audit at the same time

```
Kafka Message Size
The optimal size of a Kafka message is 10KB or less. Any publishing applications should try to keep this payload within these limit.
The AVRO compression can help reducing the size of the payload.
The smaller the size the faster a message can be published and consumed off a topic.
```

- Feeding Redis Cache From Kafka
  - From Bulk Feed Load Kafka Topic
    - Add
      - Insert Record
        - Record Key already exists then overwrite
  - From Change Kafka Topic(near realtime)
    - Update or Delete
      - Add
        - Insert Record
          - Record Key already exists then overwrite
      - Update
        - Delete Old Record
        - Insert new Record
      - Delete
        -Delete Record


![bab3a307336b3d2c408a7fc820cd5e8](https://github.com/user-attachments/assets/833de4ef-065d-42f5-b72f-8a28628ff5b0)

**Proposed Options:**

**Kafka Pub/Sub Reconciliation**

In Kafka-based systems, ensuring message delivery and consistency is crucial. Pub/Sub reconciliation is the process of verifying that all published messages are successfully received and processed by subscribers.

**Feature recommendations:**

- Use async methods to stream source system data for recon into storage that can capture changes.
- As needed normalize and enrich data, transform formats, add metadata like timestamps.
- Define rules for events to identify matches vs mismatches also categorized events vs exceptions.
- Maintain state for reconciliation decisions over time windows in storage.
- Create interactive dashboards over Kafka state stores exposing metrics and drilldowns.
- Implement reprocessing logic to take action on exceptions like replaying events, triggering notifications etc.
- Use role-based access control for ops teams to view and manage reconciliation state.

**1. Message Counts:**

- Description: Tracks the total number of messages published and subscribed to a topic.
- Advantages:
  - Simple to implement.
  - Useful for high-level monitoring and bulk reconciliation.
  - Ideal for time-series analysis of message flow.
- Disadvantages:
  - Doesn't provide granular insights into individual message status.

- **Rebalancing Errors**: rebalancing errors can occur during consumer group rebalances, which are processes where partitions are dynamically reassigned among active consumers within a consumer group.

**2. Message UUIDs:**
- Description: Tracks message delivery using unique identifiers (UUIDs).
- Requirements: Storage for UUIDs, topic/queue names, create timestamps (publish side), and update timestamps (subscribe side).
- Advantages:
  - Enables tracking of individual message delivery status.
  - Facilitates identification of delayed or missing messages within time windows.
- Disadvantages:
  - Requires additional storage for UUIDs and timestamps.

**3. Message Hash/Digest: (Phase1- Recommended Approach)**
- Description: Uses message content hashes to detect discrepancies between published and consumed messages.
- Advantages:
  - Efficiently identifies missing or altered messages.
  - Preserves privacy by avoiding storage of full message content.
- Disadvantages:
  - Doesn't provide detailed information about specific message content.
- Checksum (Optional):
  - Can be used instead of Hash/MD for efficiency and basic error detection in less sensitive data.

**4. Audit Logs: (Phase2- Recommended Approach)**
- Description: Stores full message content for comprehensive reconciliation.
- Advantages:
  - Offers the most detailed insights into message flow and content.
  - Enables reprocessing of specific messages if needed.
  - Commonly used in B2B solutions for compliance and auditing.
- Disadvantages:
  - Requires significant storage capacity for message content(payload).

**Choosing the Right Metric**

The optimal metric depends on your specific requirements:
- Bulk reconciliation: Message counts may suffice.
- Individual message tracking: UUIDs or audit logs are more suitable.
- Privacy concerns: Message hashes offer a privacy-preserving/data protection approach.
- Compliance or auditing needs: Audit logs provide the most comprehensive evidence.

**Additional Considerations**

- Error Handling: Implement strategies for handling message delivery failures and reprocessing.
- Monitoring and Alerting: Set up alerts for reconciliation discrepancies to ensure prompt action.
- Integration with Other Systems: Integrate reconciliation processes with monitoring and alerting tools for comprehensive visibility.

**Solutions:**
- To Reconcile from subscriber side, provide steps or API to reconcile on demand.
- Batch Load will be on Demand.

**Use case based flow:**
- **Use Case 1:**  Kafka Reconciliation Proposed Solution (Garbage in Garbage Out)
- **Use Case 2:**  Kafka Reconciliation (Validation) Fiserv Proposed Solution–Beyond Recon --if Consumer wants to do further reconciliation with bad data.


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

Feeding Redis Cache From Kafka
  From Bulk Feed Load Kafka Topic
    Add
      Insert Record
        Record Key already exists then overwrite
  From Change Kafka Topic(near realtime)
    Update or Delete
      Add
        Insert Record
          Record Key already exists then overwrite
      Update
        Delete Old Record
        Insert new Record
      Delete
        Delete Record
```

![bab3a307336b3d2c408a7fc820cd5e8](https://github.com/user-attachments/assets/833de4ef-065d-42f5-b72f-8a28628ff5b0)

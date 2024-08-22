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


![bab3a307336b3d2c408a7fc820cd5e8](https://github.com/user-attachments/assets/833de4ef-065d-42f5-b72f-8a28628ff5b0)

---
layout: post
title: "System design - notification system"
date: 2025-01-22
description: "System design"
tag: System Design
---

# Notification system design

## digest

When talk about notification system, below features should be in your mind:

1. **Notification Types**:

   - Email
   - SMS
   - push notification (app not in use like lock sreen message)
   - in-app notification (app in use)

2. **User preference**:

   - channel (like topic)
   - frequency (daily, weekly, monthly ...)
   - time (daytime on, night off)

3. **Template Management**: Support for dynamic templates that can be personalized based on user data.

4. **Retry-mechanism**:

   - fail message error handling, retry sending
   - DLQ (manual intervention)

5. **Scheduling & message priority**:

   - delayed or time-based notifications
   - priority message (system alert) should be processed ahead of others

6. **Analytics and Reporting**:

   - (delivered, opened, clicked)

7. **Multi-Tenancy**:
   - support differenct client (with same system but isolate data)

## non-functional requirements

1. availability: 99.99% uptime

2. latency: Low latency / eventual consistency

3. Fault tolearance

4. Data security: encrypt sensitive data, ensure GDPR

## Traffic & Storage estimation & Bandwidth

### Traffic

#### QPS

Suppose system will handle 200 million notifications every day.

During peak hours like marketing campaigns or system-wide alerts, assume there will be 10 million messages per min. The peak traffic will last 10 mins.

The peak QPS is 10 million / 60s = 166k/s

#### Message Queue

```
knowledge in system design interview:
    if message size is around 10k, generally one partition can handle 5k to 10k message per sec.
    if message size is around 1k, generally one partition can handle 50k to 100k message per sec.
```

Assume each message size is 1k, then say one partition can handle 50k messages/sec, we need 4 \* 50k > 166k, 4 partition to handle the traffic.

### Storage

Assume each message size is 1k, then 200 million message per day need 1k \* 200 million = 200GB/day.

That is 200 GB _ 365 _ 5 = 365 TB for 5 years.

### Bandwidth

for notifications, assume 50% are email, 30% are SMS, 20% are push notification. (need to consider the peak hour)

1. For Email: assume each email size 10k:

   - 166k _ 50% _ 10k = 880M/s

2. For SMS: assume each text size 1k:

   - 166k _ 30% _ 1k = 50M/s

3. For push: assume each notification size 500 bytes:
   - 166k _ 20% _ 500 = 16.67M/s

total is 880 + 50 + 16.67 = 947 M/s

947 / 125 = 7.2 Gbps

## Reference:

1. [system-design-notification-system-part-1](https://freedium.cfd/https://medium.com/dev-genius/system-design-notification-system-part-1-cf4efadf9fd2)

2. [system-design-notification-system-part-2](https://freedium.cfd/https://medium.com/dev-genius/system-design-notification-system-part-2-f5e703c746b2)

---
layout: post
title: "System design - Rate limiter"
date: 2025-04-10
description: "Rate limit"
tag: System Design
---

# Rate Limiter

a rate limiter is used to control the rate of traffic sent by a client or a
service.

## why

- prevent Dos attach to prevent prevent server overload
- reduce cost if using third party api, like retireve balance

## implement

- client side: throttle the button click rate, no guarentee (can be modified source code)

- server side: most cases implemented within API Gateway

- hard limit or soft limit

## Algorithms

- Token Bucket
- Leaking Bucket
- Fixed Window Counter
- sliding window log
- sliding window counter

`token bucket`: A token bucket is a container that has pre-defined capacity. Tokens are put in the bucket at preset rates periodically. Once the bucket is full, no more tokens are added.

- If there is enough token, process request and consume one token
- else: request is denied
  ![token bucket](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fi%2Fmkifh9c3ze4i9ir5j4mo.png)

- TokenBucket(bucketSize, RefillRate)

  - Refill rate: number of tokens put into the bucket every second

- How many bucket is needed depends on different api, if a user is allowed to make 1 post per second, add 150 friends per day, the we need 2 buckets

  ```text
  Pros:
      1. easy to implement
      2. memory efficient
  Cons:
      1. hard to tune the params properly
  ```

`Leaking bucket`: FIFO queue

- if queue is not full, add it to queue - else drop it
- Requests are pulled from the queue and processed at regular intervals.

- leakingBucket(BucketSize, OutflowRate):

  - Outflow rate: it defines how many requests can be processed at a fixed rate, usually in seconds.

  ```text
  Pros:
      1. memory efficient given the limited queue size
      2. Requests are processed at a fixed rate therefore it is suitable for use cases that a stable outflow rate is needed.
  Cons:
      1. when queue filled with old requests, burst recent traffic will be dropped
      2. hard to tune params
  ```

`Fixed window counter`

- The algorithm divides the timeline into fix-sized time windows and assign a counter for each window
- Each request increments the counter by one - Once the counter reaches the pre-defined threshold, new requests are dropped until a new time window starts.

- issue: [0.5 : 1] + [1, 1.5] this one second can process 2 \* count requests

  ```text
  Pros:
      1. memory efficient
      2. fit certain cases
  Cons:
      issue: [0.5 : 1] + [1, 1.5] this one second can process 2 * count requests
  ```

`sliding window log` : fix the issue of fixed window counter

- keep track request timestamp in cache (redis)
- when new request came, remove all outdated timestamp, eg request comes at 1:01:30, so the frame is [1:00:30 - 1:01:30], remove all timestamp before 1:00:30 - add timestamp to log - if log size <= allowed count, process else rejected

  ```text
  Pros:
      1. accurate rate limit, in any rolling window, request will not exceed limit
  Cons:
      1. need memory to store request timestamp
  ```

`sliding window counter`: combine fixed window counter and sliding window log

- Requests in current window + requests in the previous window \* overlap percentage of the rolling window and previous window
- round up or down, then compare with allowed count

  ```text
  Pros:
      1.  It smooths out spikes in traffic because the rate is based on the average rate of the previous window
      2. memory efficient
  Cons:
      1. not so strict
  ```

## [implementation Code with Redis](https://systemsdesign.cloud/SystemDesign/RateLimiter)

## High level architure

in-memory cache like Redis is a good choice, as this is time-based expiration strategy.

![high level](https://thealgoristsblob.blob.core.windows.net/thealgoristsimages/rate-limiter-sys-design-3.jpeg)

## Deep Dive

`rules`:

```text
domain: auth
descriptors:
  - key: auth_type
    Value: login
    rate_limit:
      unit: minute
      requests_per_unit: 5
```

rules often stored on configs or saved on disk. generally rules will be cached as well.

`Headers`:
client can get information from headers about the rate limit.

- X-Ratelimit-Remaining: The remaining number of allowed requests within the window.
- X-Ratelimit-Limit: It indicates how many calls the client can make per time window.
- X-Ratelimit-Retry-After: The number of seconds to wait until you can make a request again without being throttled.

When a user has sent too many requests, a 429 too many requests error and X-Ratelimit-Retry-After header are returned to the client.

## Rate limiter in distributed systems

In distributed systems, generally it will have two problems:

- `Race condition`:

  - `Lua script`
  - `sorted set`

- `Sync problem`:
  - same client to same limiter (not scalable)
  - better solution:
    - use centralized data stores like Redis, all rate limiter fetch data from Redis

## performance

- set up multi-data center to reduce latency
- sync data in eventual consistency model (trade strict limit for latency)

## metric & monitor

- `limit algorithm is effective`
- `limit rules are effective`

  ```
  too many valid request are dropped -> relax rule
  flash sales, burst traffic -> change to token buckets
  ```

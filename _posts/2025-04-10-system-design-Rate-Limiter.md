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
- ![token bucket](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fi%2Fmkifh9c3ze4i9ir5j4mo.png)

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
    - if queue is not full, add it to queue
    - else drop it
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
    - Each request increments the counter by one
    - Once the counter reaches the pre-defined threshold, new requests are dropped until a new time window starts.

issue: [0.5 : 1] + [1, 1.5] this one second can process 2 * count requests

```text
Pros:
    1. memory efficient
    2. fit certain cases
Cons:
    issue: [0.5 : 1] + [1, 1.5] this one second can process 2 * count requests
```

`sliding window log` : fix the issue of fixed window counter
    - keep track request timestamp in cache (redis)
    - when new request came, remove all outdated timestamp, eg request comes at 1:01:30, so the frame is [1:00:30 - 1:01:30], remove all timestamp before 1:00:30
    - add timestamp to log
    - if log size <= allowed count, process else rejected

```text
Pros:
    1. accurate rate limit, in any rolling window, request will not exceed limit
Cons:
    1. need memory to store request timestamp
```

`sliding window counter`: combine fixed window counter and sliding window log
    - Requests in current window + requests in the previous window * overlap percentage of the rolling window and previous window
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

too many valid request are dropped -> relax rule
flash sales, burst traffic -> change to token buckets

## Improvement with Bloom Filters & Count-Min Sketches

[Reference](https://freedium.cfd/https://levelup.gitconnected.com/system-design-rate-limiting-system-with-bloom-filters-f540f19152ef)

Use `Bloom Filter` to track requests and estimate whether a user has made requests in the current time window.

Use `Count-min Sketches` to count the number of requests for each user, ensuring a precise count within the time window

`Bloom Filter` Class Implementation

```Java
import java.util.BitSet;

public class BloomFilter {
    private BitSet bitSet;
    private int size;
    private int[] hashSeeds; // List of seeds for different hash functions

    public BloomFilter(int size, int[] hashSeeds) {
        this.size = size;
        this.bitSet = new BitSet(size);
        this.hashSeeds = hashSeeds;
    }

    // Insert an item (e.g., User ID) into the Bloom Filter
    public void add(String item) {
        for (int seed : hashSeeds) {
            int hash = hash(item, seed);
            bitSet.set(Math.abs(hash % size));
        }
    }

    // Check if an item exists in the Bloom Filter
    public boolean mightContain(String item) {
        for (int seed : hashSeeds) {
            int hash = hash(item, seed);
            if (!bitSet.get(Math.abs(hash % size))) {
                return false; // If any bit is not set, item is definitely not present
            }
        }
        return true; // Otherwise, item might be present
    }

    // Hashing function
    private int hash(String item, int seed) {
        return item.hashCode() * seed;
    }

    // Reset the Bloom filter for a new time window
    public void reset() {
        bitSet.clear();
    }
}
```

`Count-min Sketches` Implementation

```Java
import java.util.Random;

public class CountMinSketch {
    private int[][] sketch;
    private int depth;
    private int width;
    private int[] hashSeeds;

    public CountMinSketch(int depth, int width) {
        this.depth = depth;
        this.width = width;
        this.sketch = new int[depth][width];
        this.hashSeeds = new int[depth];

        // Initialize hash seeds for each row
        Random random = new Random();
        for (int i = 0; i < depth; i++) {
            hashSeeds[i] = random.nextInt();
        }
    }

    // Add an item and increment its count
    public void add(String item) {
        for (int i = 0; i < depth; i++) {
            int hash = hash(item, hashSeeds[i]);
            sketch[i][hash % width] += 1;
        }
    }

    // Estimate the count of an item
    public int estimateCount(String item) {
        int min = Integer.MAX_VALUE;
        for (int i = 0; i < depth; i++) {
            int hash = hash(item, hashSeeds[i]);
            min = Math.min(min, sketch[i][hash % width]);
        }
        return min;
    }

    // Hashing function
    private int hash(String item, int seed) {
        return item.hashCode() * seed;
    }

    // Reset Count-Min Sketch for a new time window
    public void reset() {
        for (int i = 0; i < depth; i++) {
            for (int j = 0; j < width; j++) {
                sketch[i][j] = 0;
            }
        }
    }
}
```

`Rate Limiter Class` Implementation

```Java
public class RateLimiter {
    private BloomFilter bloomFilter;
    private CountMinSketch countMinSketch;
    private int maxRequests; // Max allowed requests per user/IP per time window

    public RateLimiter(int bloomFilterSize, int[] hashSeeds, int depth, int width, int maxRequests) {
        this.bloomFilter = new BloomFilter(bloomFilterSize, hashSeeds);
        this.countMinSketch = new CountMinSketch(depth, width);
        this.maxRequests = maxRequests;
    }

    // Process a request from a user and check if it's within the rate limit
    public boolean isRequestAllowed(String userId) {
        if (bloomFilter.mightContain(userId)) {
            // Check exact count in Count-Min Sketch
            int currentCount = countMinSketch.estimateCount(userId);
            if (currentCount >= maxRequests) {
                return false; // Exceeds rate limit
            }
        }

        // If not exceeded, add the request
        bloomFilter.add(userId);
        countMinSketch.add(userId);
        return true; // Request allowed
    }

    // Reset for new time window
    public void reset() {
        bloomFilter.reset();
        countMinSketch.reset();
    }
}
```

`Window Manager` Class Implementation

```Java
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class WindowManager {
    private RateLimiter rateLimiter;
    private int windowDuration; // Duration in seconds for each sliding window

    public WindowManager(RateLimiter rateLimiter, int windowDuration) {
        this.rateLimiter = rateLimiter;
        this.windowDuration = windowDuration;
    }

    public void startWindowRotation() {
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
        
        // Schedule BloomFilter and CountMinSketch reset at regular intervals
        scheduler.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                rateLimiter.reset();
            }
        }, windowDuration, windowDuration, TimeUnit.SECONDS);
    }
}
```

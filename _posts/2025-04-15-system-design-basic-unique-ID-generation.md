---
layout: post
title: "System design basic - Unique ID generator"
date: 2025-04-15
description: "ID generation"
tag: System Design
---

## Common way to generate Unique ID in distributed system

In distributed system, it is not enough to rely on auto_increment provided by a transactional database.

`multi-master replication`: 

Suppose we have k databases, each database increase by k, there will be no conflicts amoung IDs.

```text
Pros:
    1. easy to implement
    2. solve some scalability issues
Cons:
    1. not scale well when new db added or removed.
    2. not time sensitive
```

`UUID`:

UUID is a 128-bit number unique ID. Can be used to generate ID independently without coordination between servers

```text
Pros:
    1. easy to use
    2. easy to scale because do not need to synchronizae ID with other servers
Cons:
    1. fixed 128-bits long
    2. not time sensitive
    3. could be non-numeric
```

`Ticket server`:

Ticket server basiclly is a single Ticket server responsible for the ID generation, before insert into database, call Ticket server to get the unique id then store it.

```text
pros:
    1. ID type can be customized.
    2. easy to implement and works fine with medium-scale applications.
Cons:
    1. single point of failure. If using multiple ticket server will introduce synchronization problems
```

`snowflake`:

Twitter's ID generation system called snowflake. It divide 64 bits into below:

**| sign (1 bit) | timestamp (41 bit) | datacenter ID (5 bit) | worker ID (5 bit) | sequence ID (12 bit) |**

```text
sign is always 0 to represent positive number.
machine ID inclue datacenter ID + worker ID
sequence ID: For every ID generated on that machine/process, the sequence number is incremented by 1. The number is reset to 0 every millisecond.
In theory, a machine can support a maximum of 4096 new IDs per millisecond.
```

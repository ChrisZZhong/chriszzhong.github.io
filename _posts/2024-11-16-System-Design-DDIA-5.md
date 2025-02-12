---
layout: post
title: "System design DDIA digest - 5. Replication"
date: 2024-11-16
description: "System design"
tag: System Design
---

# 5. Replication

Replication means: keep a copy of data on multiple machines.

- **geographicly close to users to reduce latency**
- **increase availability even if one machine is down**
- **increase read throughput**

replication algorithm:

- single-leader replication
- multi-leader replication
- leaderless replication

Type of replication:

- synchronous vs asynchronous

## Leader-follower (master-slave) replication

<img src="/images/System-Design/DB_replication.png">

One node is the master, others are slaves. **Master node is the only node that can accept write requests.** When a write request comes in, it is first written to the master local storage, master send the changes as part of **Replication log or change stream** to all slave nodes. Slave nodes take the changes and apply them to their local storage.

summary:
write to master, read from master or slave

![image](https://github.com/user-attachments/assets/83b2b00f-d05f-488b-9131-ae0f9dd88d35)

Benefit:

- Better performance: multi slave nodes support parallel read operation.
- Reliability: data is replicated across multiple locations, do not need to worry about data loss when one machine is down or destroyed
- High availability: master or slave down, DB can still be functional

When slave down, master take its job (if other slaves not available), new slave node take the job back, replace old node
When master down, a slave promote to master, a new slave node will take old slave node job

### synchronous vs asynchronous

![image](https://github.com/user-attachments/assets/d08bf40b-4c6f-4eca-94b9-09c36e2db37d)

In the example of Figure 5-2, the replication to follower 1 is synchronous: the leader waits until follower 1 has confirmed that it received the write before reporting success to the user, and before making the write visible to other clients. The replication to follower 2 is asynchronous: the leader sends the message, but doesn’t wait for a response from the follower.

```
Synchronous:
  advantage: if master down, slave node still have up-to-date data, increase read availability.
  disadvantange: if slave do not respond or down, the write will halt.
```

Async vice versa

```
Asynchronous:
  advantage: do not need to wait the response from follower, still can write when master down.
  disadvantange:
    1. if master down, the slave might not have up-to-date data to read.
    2. if salve down or not respond, write still can be processed, but not durable, it might lose data
  usage: often used when follower are geographically distributed.
```

**In practice, generally one of the followers is synchronous, and the others are asynchronous. If the synchronous follower becomes unavailable or slow, one of the asynchronous followers is made synchronous. This guarantees that you have an up-to-date copy of the data on at least two nodes: : the leader and one synchronous follower. This configuration is sometimes also called semi-synchronous**

### Set up new followers

setting up a follower can usually be done without downtime.

1. Take a consistent snapshot of the leader’s database at some point in time—if possible, without taking a lock on the entire database.
2. Copy the snapshot to the new follower node.
3. The follower connects to the leader and requests all the data changes that have happened since the snapshot was taken.
4. When the follower has processed the backlog of data changes since the snapshot, we say it has caught up

### Handling Node Outages

#### follower failure

If follower failed, it can easily catch up by requesting the recent changes from leader node's log.

#### leader failure

1. to determine the leader is failed, generally use **time out like 30s no respond**
2. **choose a new leader** Getting all the nodes to agree on a new leader is a consensus problem
3. Reconfiguring the system to use the new leader. If the old leader comes back, it might still believe that it is the leader, not realizing that the other replicas have forced it to step down. The system needs to ensure that the old leader becomes a follower and recognizes the new leader. **it could happen that two nodes both believe that they are the leader. This situation is called split brain, and it is dangerous: if both leaders accept writes, and there is no process for resolving conflicts**

## Replication Lag

### Statement based replication

**Statement based replication**: the leader logs the original SQL queries that made the changes, and the followers execute those queries to make the same changes to their copy of the database.

```
Should be careful with following cases:
1. Be careful with non-deterministic queries, like `NOW()` or `RAND()`, which will cause different results on different machines. ( replace any nondeterministic function calls with a fixed return value)

2. auto-incrementing columns: if the leader generates a new ID for a row, the follower must generate the same ID, or the row will have a different ID on the follower. (not good for concurrent write)

3. triggers, stored procedures, and user-defined functions may result in different behavior on the leader and follower.
```

New version of MySQL and PostgreSQL use **row-based replication** instead of statement-based replication. In row-based replication, the leader sends the changed rows to the followers, rather than the original SQL queries.

### Write-Ahead Log (WAL) shipping

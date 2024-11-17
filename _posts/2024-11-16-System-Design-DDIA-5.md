---
layout: post
title: "System design DDIA digest - 5. Replication"
date: 2024-11-16
description: "System design"
tag: OOD & System Design
---

## 5. Replication

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

### Leader-follower (master-slave) replication

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

setting up a follower can usually be done without downtime
![image](https://github.com/user-attachments/assets/b9ac986a-9b59-4782-acfb-583ba6dc1824)




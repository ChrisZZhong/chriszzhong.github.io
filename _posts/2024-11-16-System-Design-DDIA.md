---
layout: post
title: "System design DDIA digest"
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

### Database Replication - master slave

<img src="/images/System-Design/DB_replication.png">

One node is the master, others are slaves. **Master node is the only node that can accept write requests.** When a write request comes in, it is first written to the master local storage, master send the changes as part of **Replication log or change stream** to all slave nodes. Slave nodes take the changes and apply them to their local storage.

summary:
write to master, read from master or slave

Benefit:

- Better performance: multi slave nodes support parallel read operation.
- Reliability: data is replicated across multiple locations, do not need to worry about data loss when one machine is down or destroyed
- High availability: master or slave down, DB can still be functional

When slave down, master take its job (if other slaves not available), new slave node take the job back, replace old node
When master down, a slave promote to master, a new slave node will take old slave node job

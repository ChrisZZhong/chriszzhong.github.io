---
layout: post
title: "System design Basic - scale up system"
date: 2024-05-03
description: "System design"
tag: OOD & System Design
---

## Basic concepts

### Database Replication - master slave

<img src="/images/System-Design/DB_replication.png">

In this structure, master node only responsible for the write operation, Several slave nodes responsible for the read operations.

Benefit:
- Better performance: multi slave nodes support parallel read operation.
- Reliability: data is replicated accross multiple locations, do not need worriy about data loss when one machine is down or destroyed
- High availability: master or slave down, DB can still be functional

When slave down, master take its job (if other slaves not available), new slave node take the job back, replace old node 
When master down, a slave promote to master, a new slave node will take old slave node job

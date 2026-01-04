---
layout: post
title: "UB System Design Prep"
date: 2025-12-19
description: "UB SD"
tag: Algorithms
prime: false
---

## [programhelp](https://programhelp.net/?s=uber)

### 设计一个类似 Uber 的 ride-sharing platform，支持全球范围的乘客和司机匹配

思路：构建一个低延迟、高可用的分布式系统。首先，系统需要解决的核心难点是位置服务，可以使用GeoHash 或者 Quadtree 来快速存储和查询附近司机。然后，系统架构采用微服务设计，把用户、位置、匹配、行程、支付等服务解耦，通过API网关统一调度，并利用消息队列实现服务间的异步通信，保证系统的高扩展性和容错能力。最后进行全球部署和数据分片，例如按城市或区域对数据库和缓存进行分片，并通过多区域数据中心部署降低延迟，同时利用动态定价算法来实时调节供需平衡。

### 设计一个类似 ChatGPT 的系统

他当时对面试官讲的主线大概是：
API 和请求结构先定义，明确 sync vs async
Inference server 如何分片 / 分模型权重
Token streaming 的实现方式
Model server 前加 queuing & batching 提升吞吐
Cache（embedding / previous conversation context）怎么做
Bottleneck：GPU saturation、context length、负载波峰
解决方案：dynamic batching、multi-cluster、failover
整个 45 分钟他完全主导节奏，面试官问的每个追问都能顺着推下去。

我只在他确认 requirement 时轻声提醒了两句：“提一下 P99 latency target”、“问一下 multi-region”。

这两句问出来以后，面试官的态度明显变得积极很多。

---
layout: post
title: "System Design 方法论索引 - 快速导航"
date: 2025-12-17
description: "FAANG系统设计面试方法论快速导航和索引"
tag: System Design
prime: false
---

# System Design 方法论索引
## System Design Methodology Index

> **快速导航：根据题目类型快速找到对应的方法论**
> **Quick Navigation: Quickly find the right methodology based on problem type**

---

## 📚 方法论列表
### Methodology List

### 1. [Search 方法论](./2025-12-17-system-design-search-methodology.md)
### Search Methodology

**适用题目：**
**Applicable Problems:**
- Design Facebook Post Search
- Design Google Search
- Design Twitter Search
- 任何涉及"搜索"的题目

**核心决策：**
**Core Decision:**
- 什么时候需要深入实现 Inverted Index？
- When to implement Inverted Index from scratch?
- 什么时候可以用 Elasticsearch 快速带过？
- When to use Elasticsearch (quick pass)?

**关键要点：**
**Key Points:**
- Inverted Index 实现
- Tokenization 处理
- 多关键词查询
- Elasticsearch 快速方案

---

### 2. [Aggregation/Analytics 方法论](./2025-12-17-system-design-aggregation-methodology.md)
### Aggregation/Analytics Methodology

**适用题目：**
**Applicable Problems:**
- Design YouTube Top K
- Design Ad Click Aggregator
- Design Metrics System
- Design Dashboard Analytics
- 任何涉及"聚合"、"统计"、"Top K"的题目

**核心决策：**
**Core Decision:**
- 流处理 vs 批处理？
- Stream processing vs batch processing?
- 精确聚合 vs 近似聚合？
- Exact aggregation vs approximate aggregation?
- Tumbling Windows vs Sliding Windows？

**关键要点：**
**Key Points:**
- Flink/Spark Streaming
- 时间窗口处理
- 预计算策略
- 存储优化

---

### 3. [Matching/Real-time Systems 方法论](./2025-12-17-system-design-matching-realtime-methodology.md)
### Matching/Real-time Systems Methodology

**适用题目：**
**Applicable Problems:**
- Design Uber ⭐ **Uber 面试重点**
- Design Lyft
- Design Food Delivery (DoorDash)
- Design Real-time Location Tracking
- 任何涉及"匹配"、"实时"、"地理位置"的题目

**核心决策：**
**Core Decision:**
- Geospatial 查询选择（Redis Geo vs PostGIS）
- WebSocket vs SSE vs Long Polling
- 匹配算法选择（最近邻 vs 评分系统）

**关键要点：**
**Key Points:**
- Redis GeoHash
- WebSocket 连接管理
- 匹配算法优化
- 实时位置更新

---

### 4. [Storage/Database Systems 方法论](./2025-12-17-system-design-storage-database-methodology.md)
### Storage/Database Systems Methodology

**适用题目：**
**Applicable Problems:**
- Design URL Shortener
- Design Key-Value Store
- Design Distributed Cache
- Design File Storage System
- Design Pastebin
- 任何主要关注"存储"、"数据库"的题目

**核心决策：**
**Core Decision:**
- SQL vs NoSQL vs Key-Value Store？
- 分片策略（Range/Hash/Directory）
- 一致性策略（强一致 vs 最终一致）
- 缓存策略（Cache-Aside/Write-Through）

**关键要点：**
**Key Points:**
- 数据库选择
- 分片策略
- 一致性模型
- 缓存策略

---

### 5. [Rate Limiting/Throttling 方法论](./2025-12-17-system-design-rate-limiting-methodology.md)
### Rate Limiting/Throttling Methodology

**适用题目：**
**Applicable Problems:**
- Design Rate Limiter
- Design API Gateway
- Design Throttling System
- Prevent DDoS Attack
- 任何涉及"限流"、"节流"、"API Gateway"的题目

**核心决策：**
**Core Decision:**
- 限流算法选择（Fixed/Sliding/Token Bucket）
- 单机 vs 分布式限流
- 精确限流 vs 近似限流

**关键要点：**
**Key Points:**
- 限流算法（5 种主要算法）
- Redis 分布式实现
- 一致性保证
- 故障降级

---

### 6. [Messaging/Communication Systems 方法论](./2025-12-17-system-design-messaging-communication-methodology.md)
### Messaging/Communication Systems Methodology

**适用题目：**
**Applicable Problems:**
- Design WhatsApp
- Design Chat System
- Design Notification System
- Design Messenger
- 任何涉及"消息"、"聊天"、"通知"的题目

**核心决策：**
**Core Decision:**
- WebSocket vs SSE vs Long Polling
- 消息存储策略（在线/离线/历史）
- 消息顺序保证
- 消息可靠性保证

**关键要点：**
**Key Points:**
- WebSocket 连接管理
- 消息存储分层
- 消息队列（Kafka）
- 推送通知

---

### 7. [Driver Heat Map 方法论](./2025-12-17-system-design-driver-heatmap-methodology.md) ⭐ **Uber 高频题**
### Driver Heat Map Methodology

**适用题目：**
**Applicable Problems:**
- Design Driver Heat Map
- Design Real-time Heat Map
- Design Location Density Visualization
- 任何涉及"热力图"、"密度聚合"的题目

**核心决策：**
**Core Decision:**
- 网格聚合 vs K-D Tree vs 分层聚合 vs 近似算法
- Geohash Precision 选择
- 实时性 vs 精度权衡
- 更新频率 vs 查询延迟

**关键要点：**
**Key Points:**
- Geohash 网格划分
- Flink 流处理聚合
- Redis 多级存储
- 方案对比和 Trade-offs

---

## 🎯 快速决策指南
### Quick Decision Guide

### 根据题目名称判断
### Judge by Problem Name

```
题目包含 "Search"
Problem contains "Search"
→ [Search 方法论](./2025-12-17-system-design-search-methodology.md)
→ 判断：深入实现 vs Elasticsearch

题目包含 "Top K" / "Analytics" / "Aggregator"
Problem contains "Top K" / "Analytics" / "Aggregator"
→ [Aggregation 方法论](./2025-12-17-system-design-aggregation-methodology.md)
→ 判断：流处理 vs 批处理

题目包含 "Uber" / "Matching" / "Real-time" / "Location"
Problem contains "Uber" / "Matching" / "Real-time" / "Location"
→ [Matching/Real-time 方法论](./2025-12-17-system-design-matching-realtime-methodology.md)
→ 判断：Geospatial 查询、WebSocket

题目包含 "URL" / "Shortener" / "Storage" / "Cache"
Problem contains "URL" / "Shortener" / "Storage" / "Cache"
→ [Storage/Database 方法论](./2025-12-17-system-design-storage-database-methodology.md)
→ 判断：数据库选择、分片策略

题目包含 "Rate Limiter" / "API Gateway" / "Throttling"
Problem contains "Rate Limiter" / "API Gateway" / "Throttling"
→ [Rate Limiting 方法论](./2025-12-17-system-design-rate-limiting-methodology.md)
→ 判断：限流算法、分布式实现

题目包含 "Chat" / "WhatsApp" / "Notification" / "Messaging"
Problem contains "Chat" / "WhatsApp" / "Notification" / "Messaging"
→ [Messaging/Communication 方法论](./2025-12-17-system-design-messaging-communication-methodology.md)
→ 判断：实时通信、消息存储

题目包含 "Heat Map" / "Heatmap" / "Density" / "Driver Map"
Problem contains "Heat Map" / "Heatmap" / "Density" / "Driver Map"
→ [Driver Heat Map 方法论](./2025-12-17-system-design-driver-heatmap-methodology.md)
→ 判断：网格聚合、方案对比
```

### 根据核心需求判断
### Judge by Core Requirements

```
需要"关键词查找" → Search
需要"聚合统计" → Aggregation
需要"实时匹配" → Matching/Real-time
需要"存储数据" → Storage/Database
需要"限制请求" → Rate Limiting
需要"传递消息" → Messaging/Communication
需要"热力图" / "密度聚合" → Driver Heat Map
```

---

## 📋 Uber SDE2 面试重点
### Uber SDE2 Interview Focus

### 最可能出现的题目类型
### Most Likely Problem Types

1. **Matching/Real-time Systems** ⭐⭐⭐
   - Design Uber（核心业务）
   - Design Real-time Location Tracking
   - Design Driver-Passenger Matching

2. **Driver Heat Map** ⭐⭐⭐
   - Design Driver Heat Map（高频题）
   - Design Real-time Heat Map
   - Design Location Density Visualization

3. **Aggregation/Analytics** ⭐⭐
   - Design Analytics Dashboard
   - Design Metrics System
   - Design Top K（热门路线）

4. **Rate Limiting** ⭐⭐
   - Design API Gateway
   - Design Rate Limiter
   - 保护系统不被过载

5. **Storage/Database** ⭐
   - Design Distributed Cache
   - Design Key-Value Store
   - 作为其他系统的组件

### 准备优先级
### Preparation Priority

```
高优先级（必须掌握）：
High Priority (Must Master):
1. Matching/Real-time Systems
   - Geospatial queries (Redis Geo)
   - WebSocket 连接管理
   - 匹配算法

2. Driver Heat Map ⭐ 新增
   - Geohash 网格聚合
   - Flink 流处理
   - 方案对比和 Trade-offs

3. Aggregation/Analytics
   - 流处理（Flink）
   - 时间窗口
   - 预计算

中优先级（重要）：
Medium Priority (Important):
4. Rate Limiting
   - 限流算法
   - 分布式实现

5. Storage/Database
   - 分片策略
   - 一致性模型

低优先级（了解即可）：
Low Priority (Good to Know):
6. Search
7. Messaging/Communication
```

---

## 🚀 面试前快速检查
### Pre-Interview Quick Check

### 5 分钟快速复习
### 5-Minute Quick Review

```
1. 看题目名称 → 判断类型
   Check problem name → Identify type

2. 查阅对应方法论 → 找到决策树
   Check corresponding methodology → Find decision tree

3. 快速浏览关键要点 → 回忆核心概念
   Quick review key points → Recall core concepts

4. 准备开场话术 → 明确需求
   Prepare opening script → Clarify requirements
```

### 面试中快速参考
### Quick Reference During Interview

```
遇到瓶颈时：
When stuck:
1. 回到方法论 → 查看优化策略
   Go back to methodology → Check optimization strategies

2. 查看设计决策点 → 找到权衡
   Check design decision points → Find trade-offs

3. 参考实战模板 → 使用话术
   Reference practical templates → Use scripts
```

---

## 📝 方法论对比总结
### Methodology Comparison Summary

| 类型 / Type | 核心挑战 / Core Challenge | 关键技术 / Key Tech | Uber 相关性 / Uber Relevance |
|------------|-------------------------|-------------------|----------------------------|
| **Search** | Inverted Index vs Elasticsearch | Inverted Index, Tokenization | ⭐ Low |
| **Aggregation** | 流处理 vs 批处理 | Flink, Time Windows | ⭐⭐ Medium |
| **Matching/Real-time** | Geospatial + 实时通信 | Redis Geo, WebSocket | ⭐⭐⭐ High |
| **Driver Heat Map** | 实时地理聚合 + 方案对比 | Geohash, Flink, Redis | ⭐⭐⭐ High |
| **Storage/Database** | 分片 + 一致性 | Sharding, Consistency | ⭐ Medium |
| **Rate Limiting** | 算法 + 分布式 | Redis, Algorithms | ⭐⭐ Medium |
| **Messaging** | 实时 + 可靠性 | WebSocket, Message Queue | ⭐ Low |

---

## ✅ 最终检查清单
### Final Checklist

面试前确认：
Before interview, confirm:

- [ ] 我理解每种类型题目的特征
  I understand characteristics of each problem type
- [ ] 我知道如何快速判断题目类型
  I know how to quickly identify problem type
- [ ] 我准备好了每种类型的决策框架
  I'm ready with decision frameworks for each type
- [ ] 我记住了关键的技术选择
  I remember key technology choices
- [ ] 我准备好了开场话术
  I'm ready with opening scripts

---

**记住：灵活运用，根据题目和面试官反馈调整策略！**
**Remember: Be flexible, adjust strategy based on problem and interviewer feedback!**

**Good luck with your Uber SDE2 interview! 🚀**


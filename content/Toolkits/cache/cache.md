---
title: "全方位理解 Cache"
date: 2024-07-18
draft: false
tags: ["Toolkits", "Cache"]
ShowToc: true
---

## 1. 什么是缓存（What is a Cache）

### 内存存储（In-Memory Storage）

* 缓存是一种将数据存储在 **更快的介质（通常是内存）** 中的技术
* 本质是 **用空间换时间**
* 缓存中的数据通常不是权威数据源，权威数据仍在数据库或磁盘中

### 缓存命中 / 未命中（Cache Hit & Miss）

* **缓存命中（Hit）**：请求的数据存在于缓存中
* **缓存未命中（Miss）**：缓存中没有数据，需要访问下层存储
* 命中率（Hit Ratio）是衡量缓存效果的核心指标

---

## 2. 缓存的应用场景（Where Is Caching Used）

### CPU 缓存（L1 / L2 / L3）

* 位于 CPU 内部或附近的硬件缓存
* 利用：

  * 时间局部性
  * 空间局部性

### 页面缓存（Page Cache）

* 操作系统级缓存
* 文件读写会优先命中内存中的 Page Cache

### CDN

* 缓存静态或半静态资源（HTML / CSS / JS / 图片）
* 减少源站压力，降低网络延迟

### 缓冲区缓存（Buffer Cache）

* 对磁盘 IO 进行缓冲
* 减少频繁的小 IO 操作

### Memcached

* 轻量级内存 KV 缓存
* 不支持复杂数据结构和持久化

### Redis

* 功能丰富的内存数据存储系统
* 支持多种数据结构、持久化、复制和集群

---

## 3. 缓存部署方式（Cache Deployment）

### 进程内缓存（In-Process Cache）

* 缓存存在于应用进程内部
* 优点：访问速度最快
* 缺点：无法共享、数据不一致

### 进程间缓存（Inter-Process Cache）

* 多进程共享同一缓存实例（同一台机器）
* 需要 IPC 或共享内存机制

### 远程缓存（Remote Cache）

* 通过网络访问缓存（如 Redis）
* 支持多服务共享和水平扩展

---

## 4. 为什么需要缓存（Why Do We Need to Cache）

### 提升性能（Improve Performance）

* 减少数据库和后端服务压力
* 避免重复计算

### 降低延迟（Reduce Latency）

* 内存访问速度远快于磁盘和网络 IO

### 阿姆达尔定律（Amdahl's Law）

* 系统整体性能提升受限于不可优化部分
* 缓存应优先优化系统中的关键耗时路径

### 帕累托分布（Pareto Distribution）

* 20% 的数据产生 80% 的访问量
* 缓存非常适合热点数据场景

---

## 5. 分布式缓存（Distributed Cache）

### 取模分片（Modulus Sharding）

* 根据 `hash(key) % N` 分片
* 节点变化会导致大量缓存失效

### 范围分片（Range-Based Sharding）

* 按 Key 范围进行分片
* 容易产生数据倾斜和热点

### 一致性哈希（Consistent Hashing）

* 节点变化只影响少量 Key
* 常用于 Redis Cluster

---

## 6. 缓存替换与失效（Cache Replacement & Invalidation）

### 缓存替换策略

* **LRU（最近最少使用）**：淘汰最近最少被访问的数据
* **LFU（最不经常使用）**：淘汰访问频率最低的数据

### 缓存失效策略

#### 白名单策略（Allowlist Policy）

* 仅允许指定 Key 进入缓存

#### 写时主动失效

* 数据更新时先写数据库，再删除或更新缓存

#### 读时主动失效

* 读数据时发现不一致，主动使缓存失效

#### TTL（过期时间）

* 为缓存设置存活时间，自动过期

---

## 7. 缓存策略（Cache Strategies）

### Cache-Aside（旁路缓存）

* 应用程序显式控制缓存
* 读：缓存 → 数据库 → 写缓存
* 写：写数据库 → 删除缓存

### Read-Through

* 缓存层负责从数据库加载数据

### Write-Around

* 写操作绕过缓存，直接写数据库

### Write-Through

* 同时写缓存和数据库，保证一致性

### Write-Back

* 先写缓存，异步写数据库
* 性能高，但有数据丢失风险

---

## 8. 缓存面临的挑战（Caching Challenges）

### 惊群效应（Thundering Herd）

* 大量请求同时发生缓存未命中
* 解决方式：加锁、请求合并

### 缓存穿透（Cache Penetration）

* 请求不存在的数据，绕过缓存直达数据库
* 解决方式：布隆过滤器、缓存空值

### 大 Key（Big Key）

* 单个 Key 占用大量内存
* 可能导致阻塞和延迟抖动

### 热 Key（Hot Key）

* 某些 Key 被高频访问
* 解决方式：本地缓存、Key 拆分、多副本

### 数据一致性（Data Consistency）

* 缓存通常只能保证最终一致性
* 需要合理的失效和过期策略

---

## 9. 其他（Others）

### 监控与告警（Monitoring & Alerts）

* 缓存命中率
* QPS 与延迟
* 内存使用率
* 热 Key 监控
* 异常告警机制


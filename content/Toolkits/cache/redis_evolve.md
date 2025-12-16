---
title: "Redis 迭代历程"
date: 2024-07-20
draft: false
tags: ["Toolkits", "Cache", "Redis"]
ShowToc: true
---
![](/Toolkits/cache/redis_evolve.webp)
> 图片来源: https://bytebytego.com

---
## 2010 - 独立版 Redis（Standalone Redis）

* **Redis 1.0 发布**，架构简单，通常用作业务应用的缓存
* **特点**：数据完全存储在内存中
* **问题**：重启 Redis 会丢失所有数据，所有流量直接打到数据库

---

## 2013 - 持久化（Persistence）

* **Redis 2.8 发布**，引入持久化机制
* **RDB**：内存数据快照，定期保存数据状态到磁盘
* **AOF（Append-Only File）**：记录每条写命令，保证数据在重启后恢复

## 2013 - 主从复制（Replication）

* Redis 2.8 增加复制功能，提高可用性
* **主节点**：处理实时读写请求
* **从节点**：同步主节点数据，实现高可用

## 2013 - Sentinel

* Redis 2.8 引入 **Sentinel**，用于监控 Redis 实例
* **功能**：

  1. 监控实例状态
  2. 事件通知
  3. 自动故障转移（failover）
  4. 配置提供者（配置管理）

---

## 2015 - 集群（Cluster）

* Redis 3.0 发布，增加 Redis 集群功能
* **Redis Cluster**：分布式数据库解决方案，通过分片管理数据
* **数据分片机制**：将数据分为 16384 个槽位，每个节点负责部分槽位

---

## 后续发展

* **2017 - Redis 5.0**：增加 Stream 数据类型
* **2020 - Redis 6.0**：引入网络模块多线程 I/O

  * Redis 架构分为 **网络模块** 和 **主处理模块**
  * 网络模块曾成为系统瓶颈，多线程 I/O 提升性能


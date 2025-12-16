---
title: "全方位理解 Redis"
date: 2024-07-18
draft: false
tags: ["Toolkits", "Cache", "Redis"]
ShowToc: true
---
![](/Toolkits/cache/redis.webp)
> 图片来源: https://bytebytego.com

---
## 一、什么是 Redis

* **Redis** = Remote Dictionary Server（远程字典服务）
* **特点**：多模型数据库，延迟低至毫秒以下
* **角色**：

---

## 二、Redis 的应用

* 被众多知名公司采用：X, Pinterest, Airbnb, Uber, Slack, Instagram 等

---

## 三、Redis 如何改变数据库格局

* 支持 **内存读写 + 全量持久化**（AOF / Snapshot）
* **高可用性**：当实例失败时，可以通过 AOF 或 Snapshot 恢复数据 → **无数据丢失**

---

## 四、Redis 数据结构

* Redis 是 **KV 数据模型**，但支持多种数据类型：

  * **STRING**："Bytebytego"
  * **BITMAP**：0100001 01101101 0110100
  * **LIST**：A → B → C → E
  * **SET**：{A, B, C}
  * **HASH**：{ "e": "bytebyte", "b": "go" }

---

## 五、基本命令

| 类型      | 命令   | 示例                                 |
| ------- | ---- | ---------------------------------- |
| 设置      | SET  | SET username "adam smith"          |
| 获取      | GET  | GET username                       |
| 删除      | DEL  | DEL username                       |
| 自增      | INCR | INCR visitor_count                 |
| Hash 设置 | HSET | HSET user:1000 name "Alice" age 30 |

---

## 六、Redis 模块

* Redis 扩展模块：

  * RedisJSON
  * RedisSearch
  * RedisGraph
  * RedisBloom
  * RedisTimeSeries
  * RedisAI
  * RedisGears
  * RedisML

---

## 七、Redis 支持 Pub/Sub

* **工作流程**：

  1. 发布者（Publisher）发送消息到 Redis 通道（Channel）
  2. 订阅者（Subscriber）接收通道消息
* 发布者无需知道订阅者信息

---

## 八、Redis 应用场景

* 分布式缓存（Distributed cache）
* 会话存储（Session store）
* 消息队列（Message queue）
* 限流（Rate limiting）
* 高速数据库（High-speed database）

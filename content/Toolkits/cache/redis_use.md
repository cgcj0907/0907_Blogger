---
title: "Redis 应用场景"
date: 2024-07-18
draft: false
tags: ["Toolkits", "Cache", "Redis"]
ShowToc: true
---

![](/Toolkits/cache/redis_use.webp)
> 图片来源: https://bytebytego.com

---
## 1. 会话管理（Session）

* **用途**：在分布式系统中共享用户会话数据
* **实现方式**：Redis STRING 或 HASH 存储 session 信息

## 2. 缓存（Cache）

* **用途**：缓存对象、页面或热点数据，减少数据库压力
* **实现方式**：Redis STRING / HASH / ZSET

## 3. 分布式锁（Distributed Lock）

* **用途**：保证多个分布式服务间的资源互斥访问
* **实现方式**：Redis STRING + 设置过期时间 + 原子操作（SETNX / Lua 脚本）

## 4. 计数器（Counter）

* **用途**：统计文章点赞数、阅读量等
* **实现方式**：Redis INCR / INCRBY

## 5. 限流器（Rate Limiter）

* **用途**：限制特定用户或 IP 的请求频率
* **实现方式**：Redis STRING / HASH + TTL 或令牌桶算法

## 6. 全局 ID 生成器（Global ID Generator）

* **用途**：生成分布式系统全局唯一 ID
* **实现方式**：Redis INCR / INCRBY

## 7. 购物车（Shopping Cart）

* **用途**：存储用户购物车中的商品及数量
* **实现方式**：Redis HASH（key=用户ID，field=商品ID，value=数量）

## 8. 用户留存计算（User Retention）

* **用途**：统计用户每日登录情况，计算留存率
* **实现方式**：Redis BITMAP 或 HyperLogLog

## 9. 消息队列（Message Queue）

* **用途**：实现简单的队列消息传递机制
* **实现方式**：Redis LIST（LPUSH / RPOP）

## 10. 排行榜（Ranking）

* **用途**：对文章、用户或商品进行排名展示
* **实现方式**：Redis ZSET（有序集合，按分数排序）


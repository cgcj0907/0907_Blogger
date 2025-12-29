---
title: "SQL 执行流程"
date: 2024-03-18
draft: false
tags: ["Toolkits", "SQL"]
ShowToc: true
---

> ⚠️ 说明：不同数据库的具体实现架构不同，下图与流程展示的是**数据库系统中常见的通用设计思想**

---

## Step 1：传输层（Transport Layer）

* 客户端通过传输层协议（如 **TCP**）向数据库发送 **SQL 语句**
* 负责：

  * 网络通信
  * 请求数据的接收与响应返回

---

## Step 2：命令解析器（Command Parser）

* 接收 SQL 语句并进行：

  * **语法分析（Syntax Analysis）**
  * **语义分析（Semantic Analysis）**
* 分析完成后，生成 **查询树（Query Tree）**

  * 查询树是 SQL 语句的结构化表示

---

## Step 3：优化器（Optimizer）

* 接收查询树
* 根据统计信息和代价模型进行优化
* 生成 **执行计划（Execution Plan）**

  * 决定：

    * 使用哪个索引
    * 表的访问顺序
    * 连接方式（如 Nested Loop、Hash Join 等）

---

## Step 4：执行器（Executor）

* 接收执行计划
* 按照执行计划逐步执行 SQL
* 不直接访问磁盘，而是通过下层模块获取数据

---

## Step 5：访问方法（Access Methods）

* 提供 **具体的数据访问逻辑**
* 根据执行计划：

  * 决定如何访问数据（全表扫描、索引扫描等）
* 从存储引擎中 **读取或修改数据**

---

## Step 6：缓冲区管理器（Buffer Manager，读操作）

> 适用于 **只读查询（如 SELECT）**

* 访问方法判断 SQL 是否为 **只读语句**
* 若是 SELECT：

  * 将请求交给 **缓冲区管理器**
* 缓冲区管理器负责：

  * 优先从 **缓存（Buffer Cache）** 中查找数据
  * 若缓存未命中，则从 **磁盘数据文件** 读取数据

---

## Step 7：事务管理器（Transaction Manager，写操作）

> 适用于 **UPDATE / INSERT 等修改操作**

* 如果 SQL 是更新或插入语句：

  * 请求交给 **事务管理器**
* 事务管理器负责：

  * 事务的开启、提交、回滚
  * 维护事务状态
  * 保证事务的一致性与隔离性

---

## Step 8：锁管理器（Lock Manager）

* 在事务执行过程中：

  * 数据需要处于 **加锁状态**
* 锁管理器负责：

  * 管理各种锁（行锁、表锁等）
  * 协调并发访问
  * 防止脏读、丢失更新等问题
* 最终保证事务满足 **ACID 特性**

  * 原子性（Atomicity）
  * 一致性（Consistency）
  * 隔离性（Isolation）
  * 持久性（Durability）

---


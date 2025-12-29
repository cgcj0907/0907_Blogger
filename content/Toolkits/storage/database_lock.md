---
title: "数据库锁类型"
date: 2024-03-26
draft: false
tags: ["Toolkits", "database"]
ShowToc: true
---
## 一、常见锁类型

### 1、共享锁（Shared Lock, S Lock）

* 允许多个事务同时读取资源，但不允许修改
* 其他事务也可以对同一资源加共享锁


### 2、排他锁（Exclusive Lock, X Lock）

* 允许事务读取和修改资源
* 其他事务不能对该资源加任何锁


### 3、更新锁（Update Lock, U Lock）

* 用于事务准备更新资源时，防止死锁


### 4、模式锁（Schema Lock）

* 用于保护数据库对象的结构（如表结构）


### 5、大批量更新锁（Bulk Update Lock, BU Lock）

* 用于批量插入操作，通过减少锁数量提升性能


### 6、键范围锁（Key-Range Lock）

* 用于索引数据，防止幻读（事务已读范围内有新行插入）


### 7、行级锁（Row-Level Lock）

* 锁定表中的特定行，其他行仍可并发访问


### 8、页级锁（Page-Level Lock）

* 锁定数据库中的特定页（固定大小的数据块）
* MySQL InnoDB 默认不使用页锁，而是使用行锁；MyISAM 使用页级锁

### 9、表级锁（Table-Level Lock）

* 锁定整张表
* 实现简单，但可能显著降低并发性能


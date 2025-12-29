---
title: "数据库 类型大全"
date: 2024-03-20
draft: false
tags: ["Toolkits",  "database"]
ShowToc: true
---

## 一、关系型数据库（Relational DB）

* 数据以表格形式组织（Rows & Columns）
* 遵循严格的模式（Schema）
* 适合事务处理和结构化数据
* 例子：MySQL, PostgreSQL, Oracle

---

## 二、联机分析处理数据库（OLAP DB）

* OLAP（Online Analytical Processing）：用于数据分析和报表
* 优化复杂查询和多维分析
* 适合商业智能（BI）场景
* 例子：ClickHouse, Amazon Redshift, Google BigQuery

---

## 三、NoSQL 数据库

* 不使用传统 SQL 或严格模式
* 适合非结构化或半结构化数据
* 四种主要类型：

### 1、图数据库（Graph DB）

* 强调数据之间的关系
* 常用于社交网络、推荐系统
* 例子：Neo4j, ArangoDB

### 2、键值存储（Key-Value Store DB）

* 每个数据都有唯一键（Key）和对应的值（Value）
* 访问快速，适合缓存、会话存储
* 例子：Redis, Memcached

### 3、文档数据库（Document DB）

* 数据以文档形式存储，类似 JSON 或 BSON
* 每个文档可有不同结构，灵活存储半结构化数据
* 例子：MongoDB, CouchDB

### 4、列式数据库（Column DB）

* 数据按列而不是按行存储
* 适合分析型查询，减少 I/O，提高查询速度
* 例子：HBase, Cassandra, ClickHouse

---

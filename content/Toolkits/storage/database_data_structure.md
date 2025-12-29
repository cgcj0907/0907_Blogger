---
title: "数据库数据架构"
date: 2024-03-30
draft: false
tags: ["Toolkits",  "database"]
ShowToc: true
---

## 一、常见索引数据结构

### 1、跳表（Skiplist）

* 内存中常用索引类型
* 典型应用：Redis
* 特点：快速搜索、插入、删除

### 2、哈希索引（Hash Index）

* Map 或 Collection 的常见实现
* 特点：查找速度快，但不适合范围查询

### 3、SSTable

* 不可变的磁盘 Map 实现
* 特点：高顺序写入效率，适合持久化存储

### 4、LSM 树（Log-Structured Merge Tree）

* 结合 Skiplist 和 SSTable
* 特点：高写入吞吐量，适合写密集型系统

### 5、B 树（B-Tree）

* 基于磁盘的索引方案
* 特点：读写性能稳定，支持范围查询

### 6、倒排索引（Inverted Index）

* 用于文档索引
* 典型应用：Lucene 搜索引擎
* 特点：快速全文搜索

### 7、后缀树（Suffix Tree）

* 用于字符串模式匹配
* 特点：支持快速子串查询

### 8、R 树（R-Tree）

* 多维数据索引，如空间数据（地理坐标）
* 特点：支持最近邻查询和范围查询

---

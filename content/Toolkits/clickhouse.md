---
title: "ClickHouse 高性能数据库"
date: 2024-08-18
draft: false
tags: ["ClickHouse", "数据库", "高性能", "Toolkits"]
ShowToc: true
---

# ClickHouse 高性能数据库

ClickHouse 是一款 **开源、高性能的列式数据库**，主要用于 **大规模数据分析 (OLAP)** 和 **实时查询**。它的特点是：

- 列式存储，读取指定列非常快  
- 支持高并发写入和查询  
- 强大的聚合功能，适合报表和日志分析  
- 支持 SQL 查询，易上手  

---

## 1️. 安装与快速开始

> 可以参考官方文档或社区教程 👉[ClickHouse 官方文档](https://clickhouse.com/docs/) 
  
### 1.1 安装
```bash
curl https://clickhouse.com/ | sh
sudo ./clickhouse install
```

### 1.2 运行
```bash
$ clickhouse-client --host server

ClickHouse client version 24.12.2.29 (official build).
Connecting to server:9000 as user default.
Connected to ClickHouse server version 24.12.2.

:)
```
Specify additional connection details as necessary:

| Option | Description |
|--------|-------------|
| `--port <port>` | The port ClickHouse server is accepting connections on. The default ports are 9440 (TLS) and 9000 (no TLS). Note that ClickHouse Client uses the native protocol and not HTTP(S). |
| `-s [--secure]` | Whether to use TLS (usually autodetected). |
| `-u [--user] <username>` | The database user to connect as. Connects as the default user by default. |
| `--password <password>` | The password of the database user. You can also specify the password for a connection in the configuration file. If you do not specify the password, the client will ask for it. |
| `-c [--config] <path-to-file>` | The location of the configuration file for ClickHouse Client, if it is not at one of the default locations. See Configuration Files. |

### 1.3 退出
To exit the client, press Ctrl+D, or enter one of the following instead of a query:

* exit or exit;
* quit or quit;
* q, Q or :q
* logout or logout;

## 2. 库和表的创建

在 ClickHouse 中，数据存储的基本单元是 **数据库（Database）** 和 **表（Table）**。  
下面是创建库和表的示例：

### 2.1 创建数据库

```sql
CREATE DATABASE IF NOT EXISTS analytics;
```
* `IF NOT EXISTS` → 避免数据库已存在时报错

* `analytics` → 数据库名称，可以根据业务自定义

### 2.2 创建表
```sql
CREATE TABLE IF NOT EXISTS analytics.visits (
    user_id UInt32,
    page_id UInt32,
    timestamp DateTime
) ENGINE = MergeTree()
ORDER BY timestamp;
```
* `MergeTree()` → 高性能表引擎

* `ORDER BY timestamp` → 按时间列排序优化查询

* `IF NOT EXISTS` → 避免表已存在时报错

## 3. 数据写入与查询
此处以后撰写
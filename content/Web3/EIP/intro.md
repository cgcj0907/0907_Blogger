---
title: "EIP 笔记合集"
date: 2024-08-16
weight: 3
draft: false
tags: ["Web3", "EIP"]
featured: true
showToc: true
---

# 简介

EIP（Ethereum Improvement Proposal）是以太坊改进提案  
任何人都能提交的标准化文档  
驱动以太坊从创世区块到今天的唯一正式机制  
ERC只是EIP的应用层子集  
所有硬分叉升级Layer2变革账户抽象都源于EIP
好的，下面是不含任何表情、**按 EIP 状态与类型分类整理的中文笔记版**，适合课堂笔记、复习或写作使用。

---

## EIP 状态

EIP（Ethereum Improvement Proposal，以太坊改进提案）在提出、讨论和落地过程中，会经历以下生命周期状态

**`Idea`**（想法阶段）

* 处于预草案阶段，仅是初步构想
* 不会被记录在官方 EIP 仓库中
* 通常在论坛、Issue 或社区中进行非正式讨论

**`Draft`**（草案阶段）

* 正式进入 EIP 生命周期的第一个阶段
* 按 EIP 模板规范化后，由 EIP Editor 合并进仓库
* 处于持续开发和修改中

**`Review`**（评审阶段）

* 作者认为提案已较为成熟
* 主动请求社区或同行进行技术评审
* 重点关注规范完整性、可行性与兼容性

**`Last Call`**（最终审查阶段）

* 进入最终审查窗口，通常为 14 天
* 由 EIP Editor 指定并设置 `last-call-deadline`
* 若发现需要进行规范性修改，将退回 Review 状态

**`Final`**（最终状态）

* 成为正式标准
* 进入终态，不再进行实质性修改
* 仅允许修正勘误或补充非规范性说明

**`Stagnant`**（停滞状态）

* Draft 或 Review 状态下，连续 6 个月无实质进展
* 会被标记为停滞
* 作者或 Editor 可重新激活并移回 Draft

**`Withdrawn`**（撤回）

* 作者主动撤回提案
* 该状态具有终结性
* EIP 编号不可再次使用，重新提出需新编号

**`Living`**（持续更新）

* 特殊状态，不进入 Final
* 允许持续演进和更新
* 典型代表为 EIP-1（EIP 流程规范）

## EIP 类型

EIP 根据内容性质与影响范围分为以下几类


**Core**（核心协议类）

* 需要共识分叉（硬分叉或软分叉）
* 或与核心客户端和协议密切相关
* 示例：EIP-5、EIP-211、EIP-225

**Networking**（网络层）

* 涉及 devp2p、轻节点协议
* 以及 Whisper、Swarm 等网络协议规范
* 示例：EIP-8

**Interface**（接口规范）

* 客户端 API、RPC、ABI 等接口层标准
* 包含方法命名、调用规范等
* 通常先在 interfaces 仓库中讨论
* 示例：EIP-6

**ERC**（应用层标准）

* 应用与合约级标准
* 是开发者最常接触的一类
* 示例：

  * ERC-20（EIP-20，代币标准）
  * ERC-721（NFT）
  * EIP-137（ENS）
  * EIP-681（URI Scheme）
  * EIP-4337（账户抽象）



**Meta**（流程与治理类）

* 描述以太坊相关流程、规范和治理机制
* 不直接修改以太坊协议代码
* 通常需要社区共识
* 属于 Process EIP
* 示例：EIP-1


**Informational**（信息说明类）

* 用于说明设计思想、背景或提供指导
* 不提出新功能或强制性规范
* 不一定代表社区共识
* 实现者可以自由选择是否遵循

## 标识含义

### 文章进度说明
- **✅ 已完稿** - 文章已完成并发布
- **📝 待撰写** - 文章正在规划或撰写中

### 共识层变更标记
- **🟥 共识变化** - 需要硬分叉或修改共识规则的EIP
- **🟦 无共识变化** - 仅影响执行层的EIP（如大多数预编译和接口标准）


## Core

* [EIP-7702 EOA 代码执行交易](/web3/eip/eip-7702) ✅ `Final` 🟥
* [EIP-2718 交易类型信封：为未来交易格式铺路](/web3/eip/eip-2718) ✅ `Final` 🟦
* [EIP-1559 基础费燃烧：让ETH成为超声波货币](/eips/eip-1559) 📝 `Final` 🟥
* [EIP-4844 Proto-Danksharding：Rollup费用暴跌90%](/eips/eip-4844) 📝 `Final` 🟥
* [EIP-7514 减缓质押增长：保护去中心化](/eips/eip-7514) 📝 `Final` 🟥
* [EIP-7623 提高CALLDATA成本为Danksharding铺路](/eips/eip-7623) 📝 `Draft` 🟥
* [EIP-4895 质押提款：上海升级核心](/eips/eip-4895) 📝 `Final` 🟥
* [EIP-7002 部分提款与主动退出](/eips/eip-7002) 📝 `Review` 🟥
* [EIP-6110 链上质押存款](/eips/eip-6110) 📝 `Review` 🟥
* [EIP-7691 EOF：EVM容器化革命（2026目标）](/eips/eip-7691) 📝 `Review` 🟥
* [EIP-4444 历史数据过期机制](/eips/eip-4444) 📝 `Draft` 🟥
* [EIP-6465/6466 Blob历史过期方案](/eips/eip-6465) 📝 `Draft` 🟥
* [EIP-5656 MCOPY指令：降低内存拷贝成本](/eips/eip-5656) 📝 `Final` 🟦
* [EIP-5920 PAY opcode：原生转账新方式](/eips/eip-5920) 📝 `Draft` 🟦
* [EIP-6780 自毁仅限创建交易：提升状态管理](/eips/eip-6780) 📝 `Final` 🟦

---

## ERC

* [EIP-20 ERC-20代币标准：以太坊生态的基石](/web3/eip/eip-20) ✅ `Final` 🟦
* [EIP-2612 Permit：ERC20的无gas授权革命](/web3/eip/eip-2612) ✅ `Final` 🟦
* [EIP-721 非同质化代币标准（NFT）](/web3/eip/eip-721) ✅ `Final` 🟦
* [EIP-1155 多代币标准：游戏与市场的完美选择](/eips/eip-1155) 📝 `Final` 🟦
* [EIP-4337 账户抽象：无需更改共识层的智能合约钱包方案](/web3/eip/eip-4337) ✅ `Final` 🟦
* [EIP-7732 EIP-7702+ERC-7730安全增强版](/eips/eip-7732) 📝 `Draft` 🟥

---

## Networking

* [EIP-7594 PeerDAS：对等数据可用性采样](/eips/eip-7594) 📝 `Draft` 🟥
* （注：Data availability / Blob 历史过期相关项在 Core 列出，Networking 以 P2P/采样协议为主）

---

## Interface

* [EIP-712 类型化结构化数据签名：链上安全签名的金标准](/web3/eip/eip-712) ✅ `Final` 🟦
* [EIP-2537 BLS预编译：加速零知识证明与聚合签名](/eips/eip-2537) 📝 `Final` 🟦

---


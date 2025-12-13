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

### 文章进度说明
- **✅ 已完稿** - 文章已完成并发布
- **📝 待撰写** - 文章正在规划或撰写中

### EIP标准化状态
- `Draft` - 草案阶段，初步提案
- `Review` - 正在审查和讨论中
- `Last Call` - 最终征求意见阶段
- `Final` - 已最终确定并实施
- `Stagnant` - 长期无进展，停滞状态
- `Withdrawn` - 已撤回的提案

### 共识层变更标记
- **🟥 共识变化** - 需要硬分叉或修改共识规则的EIP
- **🟦 无共识变化** - 仅影响执行层的EIP（如大多数预编译和接口标准）

## 签名与认证协议
- [EIP-712 类型化结构化数据签名：链上安全签名的金标准](/eips/eip-712) ✅ `Final` 🟦
- [EIP-2537 BLS预编译：加速零知识证明与聚合签名](/eips/eip-2537) 📝 `Final` 🟦

## 代币标准及其扩展
- [EIP-20 ERC-20代币标准：以太坊生态的基石](/web3/eip/eip-20) ✅ `Final` 🟦
- [EIP-2612 Permit：ERC20的无gas授权革命](/web3/eip/eip-2612) ✅ `Final` 🟦
- [EIP-721 非同质化代币标准（NFT）](/web3/eip/eip-721) ✅ `Final` 🟦
- [EIP-1155 多代币标准：游戏与市场的完美选择](/eips/eip-1155) 📝 `Final` 🟦


## 交易格式与类型
- [EIP-2718 交易类型信封：为未来交易格式铺路](/web3/eip/eip-2718) ✅ `Final` 🟦
- [EIP-7702 EOA 代码执行交易](/web3/eip/eip-7702) ✅ `Final` 🟥

## 费用市场与通缩
- [EIP-1559 基础费燃烧：让ETH成为超声波货币](/eips/eip-1559) 📝 `Final` 🟥
- [EIP-4844 Proto-Danksharding：Rollup费用暴跌90%](/eips/eip-4844) 📝 `Final` 🟥
- [EIP-7514 减缓质押增长：保护去中心化](/eips/eip-7514) 📝 `Final` 🟥
- [EIP-7623 提高CALLDATA成本为Danksharding铺路](/eips/eip-7623) 📝 `Draft` 🟥

## 质押与提款
- [EIP-4895 质押提款：上海升级核心](/eips/eip-4895) 📝 `Final` 🟥
- [EIP-7002 部分提款与主动退出](/eips/eip-7002) 📝 `Review` 🟥
- [EIP-6110 链上质押存款](/eips/eip-6110) 📝 `Review` 🟥

## 账户抽象（Account Abstraction）
- [EIP-4337 账户抽象：无需更改共识层的智能合约钱包方案](/web3/eip/eip-4337) ✅ `Final` 🟦
- [EIP-7702 新账户抽象终极方案（2025进行中）](/web3/eip/eip-7702) ✅ `Final` 🟥
- [EIP-7732 EIP-7702+ERC-7730安全增强版](/eips/eip-7732) 📝 `Draft` 🟥

## EVM与执行层未来
- [EIP-7691 EOF：EVM容器化革命（2026目标）](/eips/eip-7691) 📝 `Review` 🟥
- [EIP-4444 历史数据过期机制](/eips/eip-4444) 📝 `Draft` 🟥
- [EIP-5656 MCOPY指令：降低内存拷贝成本](/eips/eip-5656) 📝 `Final` 🟦
- [EIP-5920 PAY opcode：原生转账新方式](/eips/eip-5920) 📝 `Draft` 🟦
- [EIP-6780 自毁仅限创建交易：提升状态管理](/eips/eip-6780) 📝 `Final` 🟦

## 数据可用性与Layer2
- [EIP-4844 Blob交易与Danksharding路线图](/eips/eip-4844) 📝 `Final` 🟥
- [EIP-7594 PeerDAS：对等数据可用性采样](/eips/eip-7594) 📝 `Draft` 🟥
- [EIP-6465/6466 Blob历史过期方案](/eips/eip-6465) 📝 `Draft` 🟥


---




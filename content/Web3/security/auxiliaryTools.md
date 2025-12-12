---
title: "审计辅助工具"
date: 2025-08-16
draft: false
tags: ["Web3", "security", "Toolkits"]
showToc: true
---

## EtherScan
Etherscan 是以太坊区块链浏览器，用于查询交易、合约和账户信息。  
在审计过程中可用于：
- 检查合约部署地址及交易历史
- 验证合约源代码与链上字节码一致性
- 查询 ERC-20/ERC-721 代币持有人和转账记录


## [cloc](https://github.com/AlDanial/cloc)
统计代码行数的工具，可用于：
- 了解代码规模
- 比较不同版本的代码变化
- 支持多种语言，包括 Solidity

## [notion](https://www.notion.so/)
可用于汇总和管理审计过程中的资料、笔记和报告：
- 可以建立审计任务列表
- 追踪漏洞修复进度
- 整合团队协作

## Documentation ( 需求方提供 )
审计文档部分通常包含以下内容：
- **Roles**：合约中涉及的角色定义，例如 `Owner`、`Admin`、`User` 等
- **Scope**：审计范围，包括智能合约模块、外部依赖、第三方库
- **Commit Hash (github repository)**：记录审计所使用的代码版本
- **Known Issues**：已知问题或潜在风险列表

## Solidity Metrics ( vscode 扩展 )
Solidity 代码度量工具，用于评估合约复杂度和安全性

## Note.md ( 自己撰写 )
审计笔记记录，可包含：
- 潜在漏洞说明
- 安全优化建议
- 审计过程中发现的异常行为
- 链上交互测试结果













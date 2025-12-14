---
title: "Web3 简介"
date: 2025-11-20
lastmod: 2025-11-20
weight: 1
draft: false           # false 表示发布文章
tags: ["Web3"]
featured: true       
cover:
  image: "/covers/Web3_cover.webp"   
  relative: false     
---

# Welcome to Web3!

Web3代表互联网下一个时代以区块链为核心实现去中心化用户掌控数据和资产通过智能合约自动化交互摆脱中心化平台控制

---

## security
> 👉 [安全审计 笔记合集](/web3/security/intro)

Web3安全聚焦预防黑客攻击和漏洞利用  
采用多层防御策略包括智能合约审计访问控制实时监控  
使用Slither Mythril检测重入攻击整数溢出权限问题  
最佳实践强调私钥管理多签名钱包紧急暂停机制  
2025年主要威胁包括闪贷治理攻击跨链桥漏洞钓鱼攻击  
强调安全即代码从设计阶段整合形式验证AI驱动监控  
开源库OpenZeppelin提供安全合约模板减少常见错误  
机构级项目需持续审计漏洞赏金计划构建信任

## wallet

钱包是Web3入口管理私钥和资产交互  
分为热钱包和冷钱包  
热钱包如MetaMask浏览器扩展方便dApp连接但易受网络攻击  
冷钱包包括Ledger硬件钱包和纸钱包离线存储更安全适合大额持有  
非托管钱包用户完全控制密钥  
托管钱包如Coinbase简化但依赖平台  
ERC-4337账户抽象提升用户体验支持社交恢复气体赞助  
多链钱包如Rainbow支持Layer2降低费用  
安全提示必须使用硬件签名绝不泄露种子短语定期备份

## EIP

> 👉 [EIP 笔记合集](/web3/eip/intro)

EIP是以太坊改进提案标准化平台变更  
涵盖核心协议客户端API合约规范  
任何人可提交草案经社区讨论编辑审查进入最终状态  
核心EIP需全网共识可能引发硬分叉  
如EIP-1559引入基础费市场  
ERC是EIP子集专注应用层标准  
EIP过程确保透明包容  
从草稿到最后调用再到最终实施  
EIP-4844引入blob数据大幅降低Layer2成本  
未来EIP聚焦可扩展性隐私如Verkle树



## codeLanuage
> 👉 [合约语言 笔记合集](/web3/codelanguage/intro)

智能合约语言最终编译为EVM字节码  
Solidity主流类JavaScript支持继承修饰符库生态最丰富  
Vyper Python风格故意去除继承无限循环提升可审计性安全性  
Yul中间语言用于精细气体优化  
Rust主要用于Solana非EVM链  
Ethereum生态仍以Solidity和Vyper为主  
学习Solidity推荐从Remix IDE开始  
Vyper强调人类可读代码减少攻击面  
选择语言取决于项目复杂度和安全优先级

## EVM

EVM是以太坊虚拟机去中心化计算引擎  
执行智能合约字节码采用栈基架构深度1024  
每操作码消耗气体防止滥用  
Turing完备支持任意逻辑但有气体上限避免无限循环  
EVM负责状态转换更新账户余额存储  
所有节点一致执行保证确定性隔离  
兼容链如Polygon Arbitrum共享EVM实现无缝移植  
未来升级EOF引入容器化代码提升效率  
EVM是整个以太坊生态的计算核心

## framework
> 👉 [Web3 开发框架合集](/web3/framework/intro)
Web3开发框架大幅简化合约构建测试部署  
Hardhat JavaScript基础插件化支持控制台日志自定义任务深度集成Ethers.js  
Truffle老牌套件自带Ganache本地链Mocha测试适合新手  
Foundry Rust编写超快编译铸造Anvil分叉主网用Solidity写测试支持模糊测试  
当前主流选择  
Hardhat灵活度最高  
Truffle最适合教学  
Foundry性能极致适合高级开发者  
搭配VS Code Solidity扩展开发体验最佳

## DeFi
> 👉 [DeFi 笔记合集](/web3/defi/intro)
DeFi（Decentralized Finance，去中心化金融）是构建在区块链之上的开放式金融体系，
通过智能合约实现无需信任中介的金融服务
Uniswap基于 AMM 机制的去中心化交易所，通过流动性池实现无需订单簿的代币兑换
Aave去中心化借贷协议，用户可存入资产赚取利息，或通过超额抵押进行借贷并支持闪电贷
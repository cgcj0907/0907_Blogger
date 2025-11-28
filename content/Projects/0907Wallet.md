---
title: "0907 Wallet"
date: 2025-08-16
draft: false
tags: ["Projects", "0907Wallet"]
cover:
  image: "/covers/0907Wallet.webp"  
  relative: false
ShowToc: true
---
# 项目简介

- **项目名称**：0907 Wallet
    
- **系统类型**：基于浏览器的 Web3 去中心化钱包
    
- **系统定位**：轻量化、本地化、自托管的钱包系统
- **系统代码框架**: React + Next.js
    
- **核心目标**
    
    - 提供安全可靠的钱包账户生成与管理能力
        
    - 支持基本的转账、消息签名与资产查看功能
        
    - 实现完整的本地化存储，全流程不上传私钥
        
- **非目标说明（可选）**
    
    - 不构建链下后端服务
        
    - 不提供中心化账户体系
        
    - 不托管任何用户资产


---

# 1. 代码规范

## 1.1 代码命名规范

### 1.1.1 组件命名（React 组件、页面）
- **规则**：大驼峰（PascalCase）
- **文件命名**：同名 `.tsx` 文件也必须 PascalCase
- **示例**：
  ```ts
  AccountCard.tsx        → 正确
  UserProfile.tsx        → 正确
  account-card.tsx       → 错误
  ```

### 1.1.2 接口、类型、枚举命名
- **规则**：大驼峰（PascalCase）
- **建议**：接口可加 `I` 前缀（团队统一即可），类型可加 `T` 前缀（可选）
- **示例**：
  ```ts
  interface UserInfo { ... }
  interface IUserInfo { ... }      // 也可
  type NetworkConfig = { ... }
  type TResponse<T> = { ... }
  enum ChainId { Ethereum = 1, Sepolia = 11155111 }
  ```

### 1.1.3 常量命名（全局配置、魔法值）
- **规则**：全大写 + 下划线分隔（SCREAMING_SNAKE_CASE）
- **示例**：
  ```ts
  const API_BASE_URL = "https://api.0907wallet.com";
  const DEFAULT_CHAIN_ID = 1;
  const MAX_RETRY_COUNT = 5;
  const SUPPORTED_CHAINS = [1, 5, 11155111];
  ```

### 1.1.4 普通变量、函数、属性命名
- **规则**：小驼峰（camelCase）
- **布尔值建议**：以 `is`、`has`、`should`、`can` 开头
- **示例**：
  ```ts
  const walletAddress = "0x1234...abcd";
  let isConnected = false;
  const hasBalance = balance > 0;
  
  function connectWallet() { ... }
  async function fetchUserInfo() { ... }
  ```

### 1.1.5 文件与文件夹命名
| 类型           | 命名规则           | 示例                              |
|----------------|---------------------|-----------------------------------|
| 组件 / 页面    | PascalCase          | `AccountCard.tsx`、`settings/page.tsx` |
| 工具函数       | kebab-case 或 camelCase | `storage.ts`、`format-address.ts` |
| hooks          | 以 `use` 开头       | `useWallet.ts`、`useDebounce.ts`  |
| 样式           | 与组件同名          | `AccountCard.module.css`          |
| 常量/配置      | kebab-case          | `chains.config.ts`                |

---

## 1.2 代码注释规范

### 1.2.1 文件头注释（每个文件必须）
```ts
/**
 * @file 文件简要描述（一句话）
 * @description 更详细说明（可选）
 * @author 张三（github: zhangsan）
 * @date 2025-04-05
 */
```

### 1.2.2 导出函数 / 组件 / Hook 必须写 JSDoc
```ts
/**
 * 初始化 IndexedDB 数据库（支持自动按需建表）
 *
 * @returns IndexedDB 实例
 * @example const db = await initDB();
 */
export async function initDB(): Promise<IDBPDatabase> { ... }

/**
 * 账户卡片组件（带悬停复制动画）
 *
 * @param address - 钱包地址（0x...）
 * @param name    - 可选账户别名
 */
export default function AccountCard({ 
  address, 
  name 
}: { 
  address: string;
  name?: string;
}) { ... }
```

### 1.2.3 复杂逻辑必须解释「为什么」
```ts
// 为什么关闭再升级？
// IndexedDB 只有在 versionchange 事务中才能创建 objectStore，
// 当前连接无法创建新表，必须关闭后以新版本重新打开
db.close();
```

### 1.2.4 TODO / FIXME 统一格式
```ts
// TODO: [高] 支持多签钱包（@李四，2025-07-01）
// FIXME: 快速移入移出会导致多个 setTimeout 残留
```

### 1.2.5 禁止的注释（直接打回）
```ts
// 错误示例（解释「做什么」而不是「为什么」）
const a = 1;           // 定义变量 a
setLoading(true);      // 设置 loading 为 true

// 错误：过时注释
const MAX_RETRY = 3;   // 最大重试次数 5 次（实际是 3）
```

---

## 1.3 其他强制规范（快速记忆版）

| 项目              | 要求                                               |
|-------------------|----------------------------------------------------|
| 类型优先          | 能用 TypeScript 解决的绝不写注释                   |
| props 类型        | 必须显式声明 interface/type，禁止 any              |
| 自定义 Hook       | 必须以 `use` 开头，文件名也必须是 `useXxx.ts`      |
| async 函数        | 必须 try-catch 或返回错误状态                      |
| console.log       | 上线前必须删除或改为 logger                        |
| 魔法值            | 全部提取为常量（参考 1.1.3）                       |

---



## 2. 系统功能范围（Scope）

### 2.1 基础功能

- 用户密码创建与验证
    
- 助记词生成与展示
    
- 私钥生成与导入
    
- 多账户管理
    
- 链上资产（ETH）余额查询
    
- 交易发送（转账）
    
- 基本消息签名功能


### 2.2 扩展功能（可选）

- 多语言支持（中/英文）
    
- 钱包锁定 / 解锁
    
- 钱包数据导出（助记词 / 私钥）
    
- 友好的错误反馈与基础校验
    


---

## 3. 系统整体结构（System Overview）

系统整体由以下四个主要模块组成，各模块职责清晰、边界明确，通过标准化接口进行协作：

### 3.0 系统公用模块

👉 [详细设计请见：0907 Wallet 系统公用层设计](/projects/0907wallet/public)

### 3.1 用户管理模块



### 3.2 钱包管理模块



### 3.3 区块链交互模块




---

## 4. 安全策略概述

### 4.1 数据处理原则

- 所有敏感数据仅存储在浏览器本地
    
- 系统不会上传、备份或托管任何私钥信息


### 4.2 密码策略

- 用户密码仅在本地生效
    
- 密码用于控制钱包解锁与私钥访问权限


### 4.3 交互安全性

- 引导用户及时备份助记词
    
- 转账等敏感操作需进行确认提示



---

## 5. 前端界面与使用流程概述

### 5.1 页面结构

- 欢迎 / 初始页面
    
- 密码设置页面
    
- 创建钱包 / 导入钱包页面
    
- 钱包主页（账户与资产）
    
- 转账页面
    
- 设置页面


### 5.2 UI 使用流程

- 首次使用：初始页面 → 设置密码 → 创建/导入钱包 → 进入主页
    
- 已有钱包用户：输入密码 → 解锁钱包 → 进入主页



---

## 6. 系统边界（System Boundaries）

### 6.1 系统内的内容

- 所有前端钱包业务逻辑
    
- 本地私钥生成与管理
    
- 与区块链节点的直接交互


### 6.2 系统外的内容

- 后端服务（链下后端）
    
- 数据库（链下存储）
    
- 外部钱包插件（MetaMask 等）
    
- 外部价格服务
    
- 区块链浏览器



---

## 7. 非功能性要求（NFR）概述

### 7.1 安全性

- 私钥不会通过网络传输
    
- 用户密码需受到适当保护


### 7.2 可用性

- 页面结构简洁清晰
    
- 提供必要的错误提示


### 7.3 可维护性

- 核心钱包模块与 UI 分层
    
- 模块功能边界明确


### 7.4 扩展性

- 支持未来扩展多链能力
    
- 支持新增账户或资产类型



---

## 8. 限制与已知问题（Known Limitations）

- 浏览器环境安全性依赖用户设备本身
    
- IndexedDB 数据可能因浏览器清理而丢失
    
- 私钥泄露后不可恢复



---

## 9. 后续规划（Roadmap）

### 9.1 1.0 版本：单链钱包 + 基础转账/签名功能

实现以太坊主网基础功能，包括账户管理、ETH 转账、消息签名等核心能力。

### 9.2 2.0 版本：多链支持、NFT 基础展示

扩展至多链架构，支持 BSC、Polygon 等 EVM 兼容链，并增加 NFT 资产展示功能。

### 9.3 3.0 版本：DApp 连接（Inject Provider）

实现 EIP-1193 标准 Provider 注入，支持连接外部 DApp，提供 MetaMask 兼容的交互体验。

### 9.4 3.5 版本：移动端适配

优化移动端响应式布局，增加 PWA 支持，提升移动端用户体验。

### 9.5 4.0 版本：支持硬件钱包

集成 Ledger、Trezor 等硬件钱包，提供硬件级私钥保护能力。
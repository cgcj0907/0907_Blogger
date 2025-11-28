---
title: "0907 Wallet 用户管理模块设计"
date: 2025-08-16
draft: false
tags: ["Projects", "0907Wallet"]
ShowToc: true
---

# 模块概览
👉 [模块代码仓库](https://github.com/cgcj0907/0907-Wallet/tree/main/app/userManagement)

| 项目          | 内容                                          |
|---------------|----------------------------------------------|
| 模块名称      | 用户管理（User Management）                    |
| 路由路径      | `/userManagement`                             |
| 功能范围      | 密码设置、已有钱包登录、本地加密存储             |
| 访问权限      | 未登录状态下强制跳转到此页面，已登录放行          |

* 目录结构与文件职责
```
app/userManage/
├── components/
│   ├── Login.tsx       
│   └── Register.tsx    
├── lib/
│   └── savePassword.ts  
└── page.tsx            
```

| 文件路径                     | 职责详解                                                                                   |
|------------------------------|--------------------------------------------------------------------------------------------|
| `page.tsx`                   | 1. 路由入口<br>2. 检测 IndexedDB 是否已有钱包记录<br>3. 已登录 → 跳转 `/`（首页）<br>4. 未登录 → 渲染登录/注册切换界面 |
| `components/Login.tsx`       | 纯展示组件：<br>• 密码输入解锁本地钱包<br>• “忘记密码”删除本地数据 |
| `components/Register.tsx`    | 1. 设置密码（强度校验）<br>2. 加密存储 |
| `lib/savePassword.ts`        | 核心加密逻辑：<br>• 将密码进行 SHA256 处理<br>• 存入 IndexedDB |

## 1. 详细交互流程

### 1.1 页面访问与前置检查
1. 访问任意页面 → `AuthGuard` 检测未登录 → 重定向至 `/userManagement`
2. `page.tsx` 检查 IndexedDB 无 `user:current` 数据 → 默认展示 `Register.tsx`


### 1.2 初始化密码流程
1. 首次登录/忘记密码
2. 检查 IndexedDB 有无 `user:current`, 若有则删除本地 IndexedDB 数据库用户和钱包相关内容
3. 二次确认密码加密存储至 IndexedDB

## 2. 安全设计原则

- 密码**永不**以明文形式出现在内存、网络、日志中
- 助记词/私钥**只在内存中短时存在**，解密后立即覆盖
- 所有敏感数据均使用用户密码派生的密钥进行 AES-GCM 加密存储
- 提供“一键删除本地数据”功能（对应“忘记密码”场景）


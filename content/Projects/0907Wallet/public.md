---
title: "0907 Wallet 系统公用层设计"
date: 2025-08-16
draft: false
tags: ["Projects", "0907Wallet"]
ShowToc: true
---

# 模块概览

```bash
app
├── components
│   ├── AuthGuard.tsx
│   ├── InitProvider.tsx
│   └── Logo.tsx
├── globals.css
├── layout.tsx
├── lib
    ├── auth.ts
    ├── storage.ts
    └── transform.ts
```

* 公用库( lib )
    * auth
    * hash
    * storage
* 公用组件( components )
    * AuthGuard
    * Initialer
    * Logo

## 1. 公用库设计

### auth.ts
```ts
// app/lib/auth.ts
/**
 * @file 钱包认证与账户管理核心模块
 * @description 负责钱包连接、签名验证、登录状态持久化、登出等完整认证流程
 *              支持 MetaMask、WalletConnect、Coinbase Wallet 等主流钱包
 * @author Guangyang Zhong | github: https://github.com/cgcj0907
 * @date 2025-08-16
 */

export const LOGIN_FLAG_KEY = 'isLoggedIn';
export const LOGIN_EXPIRES_KEY = 'loginExpiresAt';
const DEFAULT_EXPIRE_MINUTES = 10;

/**获取过期时间 */
export function getLoginExpiresAt(): number | null {}

/**检查登录状态是否合法 */
export function isLoggedInLocal(): boolean {}

/** 刷新过期时间（默认 +10 分钟）并确保 flag = '1' */
export function refreshLoginExpiry(minutes = DEFAULT_EXPIRE_MINUTES) {}

/** 退出登录（本地清理） */
export function logoutLocal() {}
```

### transform.ts
```ts
// @/app/lib/hash.ts
/**
 * @file 密码哈希与 ArrayBuffer/Base64 互转工具
 * @description 提供安全的密码 SHA-256 哈希（返回 Base64）和高性能的二进制 ↔ Base64 转换工具
 * @author Guangyang Zhong | github: https://github.com/cgcj0907
 * @date 2025-11-27
 */

/**
 * 将 ArrayBuffer 转换为 Base64 字符串
 *
 * 使用 Uint8Array + btoa 的经典写法，比 Buffer.toString('base64') 更兼容浏览器环境
 *
 * @param buffer 要转换的 ArrayBuffer
 * @returns Base64 编码字符串
 * @example
 * const b64 = arrayBufferToBase64(hashBuffer);
 */
export function arrayBufferToBase64(buffer: ArrayBuffer): string {}

/**
 * 将 Base64 字符串转换为 Uint8Array
 *
 * @param base64 Base64 编码的字符串
 * @returns 对应的 Uint8Array
 * @example
 * const uint8 = base64ToUint8Array(storedHash);
 */
export function base64ToUint8Array(base64: string): Uint8Array {}

/**
 * 对明文密码进行安全的 SHA-256 哈希，并返回 Base64 编码结果
 *
 * 使用 Web Crypto API（crypto.subtle），在所有现代浏览器中均为原生实现，安全且高性能。
 *
 * @param password 明文密码
 * @returns SHA-256 哈希后的 Base64 字符串（固定 44 字符）
 * @example
 * const hashed = await hashPassword('mySecret123');
 * // → "iOcJ9qP3b...=="
 */
export async function hashPassword(password: string): Promise<string> {}
```

### storage.ts
👉 [关于 IndexedDB 的使用方法请见：IndexedDB 浏览器数据库](/toolkits/indexedDB)
```ts
// app/lib/storage.ts
/**
 * @file IndexedDB 轻量封装工具（按需懒创建 objectStore）
 * @description 提供类似 localStorage 的简洁 API，支持自动升级版本创建表，避免预设 schema
 * @author Guangyang Zhong | github: https://github.com/cgcj0907
 * @date 2025-11-27
 */

import { openDB, IDBPDatabase } from 'idb';

const DB_NAME = '0907wallet-db';
const DEFAULT_STORE_NAME = 'users';

/**
 * 初始化（或打开）IndexedDB 数据库
 *
 * 数据库不存在时会自动创建版本 1，且不预创建任何 objectStore，
 * 所有表都通过 ensureStore 按需懒创建，保持最小的初始 schema。
 *
 * @returns IndexedDB 数据库实例
 * @example
 * const db = await initDB();
 */
export async function initDB(): Promise<IDBPDatabase> {}

/**
 * 确保指定名称的 objectStore 存在（内部私有函数）
 *
 * 如果表不存在，会关闭当前连接 → 版本 +1 → 在 upgrade 事务中创建表。
 * 这是 IndexedDB 的硬性限制：只有 versionchange 事务才能创建/删除 objectStore。
 *
 * @param db        当前数据库实例
 * @param storeName 要确保存在的表名
 * @returns 包含该表的最新数据库实例
 */
async function ensureObjectStoreExists(
  db: IDBPDatabase,
  storeName: string,
): Promise<IDBPDatabase> {}

/**
 * 根据 key 从指定表中读取数据
 *
 * @param key       键名
 * @param table     表名，默认为 'users'
 * @returns         对应的值，若不存在返回 undefined
 * @example
 * const user = await get('currentUser');
 */
export async function get<T = unknown>(
  key: string,
  table: string = DEFAULT_STORE_NAME,
): Promise<T | undefined> {}

/**
 * 向指定表中写入或更新数据
 *
 * @param key       键名
 * @param value     要存储的值（必须支持结构化克隆）
 * @param table     表名，默认为 'users'
 * @example
 * await set('currentUser', { address: '0x...', name: 'Alice' });
 */
export async function set<T = unknown>(
  key: string,
  value: T,
  table: string = DEFAULT_STORE_NAME,
): Promise<void> {}

/**
 * 根据 key 删除指定表中的记录
 *
 * @param key    键名
 * @param table  表名，默认为 'users'
 */
export async function del(
  key: string,
  table: string = DEFAULT_STORE_NAME,
): Promise<void> {}

/**
 * 清空指定表的所有记录
 *
 * @param table  表名，默认为 'users'
 */
export async function clear(table: string = DEFAULT_STORE_NAME): Promise<void> {}

/**
 * 获取指定表中所有 key
 *
 * @param table  表名，默认为 'users'
 * @returns      key 数组
 */
export async function keys(
  table: string = DEFAULT_STORE_NAME,
): Promise<IDBValidKey[]> {}

/**
 * 获取指定表中所有值（不包含 key）
 *
 * @param table  表名，默认为 'users'
 * @returns      值数组
 */
export async function values<T = unknown>(
  table: string = DEFAULT_STORE_NAME,
): Promise<T[]> {}

/**
 * 获取指定表的记录总数
 *
 * @param table  表名，默认为 'users'
 * @returns      记录数量
 */
export async function count(table: string = DEFAULT_STORE_NAME): Promise<number> {}
```

## 2. 公用组件设计
 
### AuthGard.tsx
```tsx
// app/components/AuthGuard.tsx
/**
 * @file 全局路由权限守卫（客户端组件）
 * @description 
 *   1. 未登录 → 强制跳转 /userLogin（公开页除外）
 *   2. 已登录但无钱包 → 强制跳转 /generateWallet
 *   3. 全部通过 → 放行并自动刷新登录有效期
 * @author Guangyang Zhong | github: https://github.com/cgcj0907
 * @date 2025-11-27
 */

'use client';

import React, { useEffect, useState } from 'react';
import { useRouter, usePathname } from 'next/navigation';
import { count } from '@/app/lib/storage';
import * as auth from '@/app/lib/auth';

interface AuthGuardProps {
  /** 要渲染的子内容（受保护的页面） */
  children: React.ReactNode;
}

// 公开路径：无需登录即可访问
const PUBLIC_PATHS = ['/userLogin', '/generateWallet'] as const;

// 钱包存储表名（避免魔法值）
const WALLETS_STORE_NAME = 'Wallets';

export default function AuthGuard({ children }: AuthGuardProps): React.ReactElement {
  const router = useRouter();
  const pathname = usePathname();
  const [isChecking, setIsChecking] = useState<boolean>(true);

  useEffect(() => {
    let isMounted = true;

    /**
     * 核心鉴权逻辑
     * 为什么放在 useEffect 里单独执行？
     * - 避免阻塞渲染
     * - 需要 async/await 必须封装在函数内
     * - 配合 isMounted 防止内存泄漏
     */
    async function performAuthCheck(): Promise<void> {
      try {
        // pathname 可能为 null（Next.js 某些边缘情况）
        if (!pathname) {
          if (isMounted) setIsChecking(false);
          return;
        }

        const isPublicPath = PUBLIC_PATHS.includes(pathname as any);

        // Step 1: 检查本地登录态（同步快速）
        const isLoggedIn = auth.isLoggedInLocal();

        if (!isLoggedIn && !isPublicPath) {
          router.replace('/userLogin');
          if (isMounted) setIsChecking(false);
          return;
        }

        // Step 2: 已登录 → 检查是否已创建钱包
        if (isLoggedIn) {
          const walletCount = await count(WALLETS_STORE_NAME);

          // count 返回 number，0 表示无钱包
          const hasWallet = walletCount > 0;

          if (!hasWallet && pathname !== '/generateWallet') {
            router.replace('/generateWallet');
            if (isMounted) setIsChecking(false);
            return;
          }
        }

        // Step 3: 全部通过 → 刷新登录有效期并放行
        if (isMounted) {
          auth.refreshLoginExpiry(); // 延长本地登录态过期时间
          setIsChecking(false);
        }
      } catch (error) {
        // 为什么 catch 后直接跳转登录？
        // IndexedDB 异常、本地存储损坏等都视为“不安全状态”，必须重新登录
        console.error('[AuthGuard] 鉴权过程中发生异常:', error);
        if (isMounted) {
          setIsChecking(false);
          router.replace('/userLogin');
        }
      }
    }

    performAuthCheck();

    // 清理函数
    return () => {
      isMounted = false;
    };
  }, [pathname, router]);

  // Loading 状态：全屏居中动效
  if (isChecking) {
    return (
      <div className="min-h-screen flex flex-col items-center justify-center bg-gray-50">
        <p className="text-gray-600 text-lg mb-4 animate-pulse">
          正在验证授权...
        </p>
        <div className="w-12 h-12 border-4 border-blue-500 border-t-transparent rounded-full animate-spin" />
      </div>
    );
  }

  // 鉴权通过：渲染子内容
  return <>{children}</>;
}
```
### InitProvider.tsx
```tsx
'use client'

import { useState } from 'react';
import clsx from 'clsx';

/**
 * @file Logo 组件（首页/导航栏主 Logo）
 * @description 带悬停交互的圆形 Logo，鼠标悬停时向上浮起并展示文字说明，支持缓慢弹跳动画
 * @author Guangyang Zhong | github: https://github.com/cgcj0907
 * @date 2025-11-27
 */

/**
 * 主 Logo 组件
 * - 默认状态：Logo 向下位移一半，遮挡下方文字区域（视觉上形成“半隐藏”效果）
 * - 悬停状态：Logo 上移至正常位置，同时下方文字卡片从 0 高度展开
 * - 使用 z-[-1]/z-[-2] 保证图片始终在文字卡片上方，但整体被外部容器裁剪
 */
export default function Logo() {
    // hover 状态控制整个交互动画的开启与关闭
    const [hover, setHover] = useState(false);

    return (
        // 外层容器：相对定位，用于控制子元素的绝对/固定定位基准，同时垂直居中排列
        <div
            className="relative flex flex-col items-center"
            onMouseEnter={() => setHover(true)}
            onMouseLeave={() => setHover(false)}
        >
            {/* 图片 Logo（主视觉元素） */}
            <div
                className={clsx(
                    // 负 z-index 让图片在文字卡片之下，但通过 translate-y 制造“浮在前面”的视觉错位
                    // w-86/h-86 ≈ 344px（Tailwind 自定义单位），圆形裁剪 + 溢出隐藏 + 阴影 + 平滑位移动画
                    "z-[-1] w-86 h-86 rounded-full overflow-hidden cursor-pointer shadow-lg transition-transform duration-300",
                    // 未悬停时向下位移 48（192px），刚好遮住下方文字卡片一半，形成悬念感
                    // 悬停时恢复原位，视觉上像“Logo 向上浮起”
                    hover ? "translate-y-0" : "translate-y-48"
                )}
            >
                {/* Logo 图片自适应铺满容器，保持原始比例不被拉伸 */}
                <img
                    src="/logo.png"
                    alt="Logo"
                    className="w-full h-full object-cover"
                />
            </div>

            {/* 文字说明卡片区域（默认完全收起，悬停时通过 max-h 展开） */}
            <div
                className={clsx(
                    // 使用 overflow-hidden + max-h 实现平滑的展开/收起动画
                    // mt-2 给卡片留出一点上边距，避免紧贴 Logo
                    "z-[-2] overflow-hidden transition-all duration-300 mt-2",
                    hover ? "max-h-40" : "max-h-0"
                )}
            >
                {/* 文字卡片本体：圆角背景 + 内边距 + 固定宽度 + 文字居中 + 缓慢弹跳动画 */}
                <div className="rounded-2xl p-3 w-40 text-center flex flex-col items-center gap-1 animate-bounce-slow">
                    {/* 主标题 */}
                    <p className="text-sm font-semibold text-blue-500 flex items-center justify-center gap-1">
                        0907 Wallet
                    </p>
                    {/* 副标题（技术支持声明） */}
                    <p className="text-xs text-blue-300">Supported by SST</p>
                </div>
            </div>
        </div>
    );
}
```
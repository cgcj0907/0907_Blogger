---

title: "0907 Wallet 钱包管理模块设计"
date: 2025-11-28
draft: false
tags: ["Projects", "0907Wallet"]
ShowToc: true
-------------

# 模块概览

👉 [模块代码仓库（walletManagement）](https://github.com/cgcj0907/0907-Wallet/tree/main/app/walletManagement)

| 项目   | 内容                                                         |
| ---- | ---------------------------------------------------------- |
| 模块名称 | 钱包管理（Wallet Management）                                    |
| 路由路径 | `/walletManagement`                                        |
| 功能范围 | 随机 HD 钱包生成、助记词展示、钱包加密/解密、加密钱包与地址本地持久化（IndexedDB）           |
| 访问权限 | 依赖本地用户密码记录（userManagement 模块）；若无用户记录则重定向 `/userManagement` |

## 目录结构与文件职责

```cosole
app/walletManagement/
├── components
│   ├── GenerateWallet.tsx
|   ├── ImportWallet.tsx
├── lib
│   ├── cryptoWallet.ts
│   ├── generateWallet.ts
|   ├── importWallet.ts
│   ├── saveAddress.ts
│   └── saveWallet.ts
├── page.tsx
└── test-cryptoWallet.ts
```

| 文件路径                            | 职责详解                                                                     |
| ------------------------------- | ------------------------------------------------------------------------ |
| `page.tsx`                      | 路由入口，渲染 GenerateWallet 组件（页面本身不包含业务逻辑）                                   |
| `components/GenerateWallet.tsx` | 核心 UI 与交互：密码校验 → 生成钱包 → 加密 → 保存 → 助记词展示；包含用户友好提示与错误处理                    |
| `lib/generateWallet.ts`         | 使用 ethers + bip39 生成随机 HD 钱包、英文与中文助记词并做一致性校验                             |
| `lib/cryptoWallet.ts`           | 提供钱包加密/解密接口，定义 `IEncryptedWallet` 类型（负责使用用户密码派生密钥对 wallet 进行 AES-GCM 加密） |
| `lib/saveWallet.ts`             | IndexedDB 封装：对 `Wallets` 表（或 store）进行保存/读取/删除/列出                         |
| `lib/saveAddress.ts`            | IndexedDB 封装：对 `Addresses` 表保存地址记录、读取、删除、列出                              |
| `test-cryptoWallet.ts`          | 单元测试：验证加密/解密、密钥派生一致性、异常场景（解密失败、篡改检测）                                     |
| `lib/importWallet.ts`           | 使用 bip39 和 ethers 从英文或中文助记词导入 HDNodeWallet（中文先转换为英文助记词再导入）                |

---

## 1. 详细交互流程

### 1.1 页面访问与前置检查

1. 用户访问 `/walletManagement`
2. `page.tsx` 渲染 `GenerateWallet`
3. 组件加载时会检查本地是否已存在用户密码记录（通过 `userManagement` 的 `getUserRecord()`）

   * 若无用户记录：重定向到 `/userManagement` 要求先设置本地密码（首次初始化）
   * 若有用户记录：继续显示生成钱包流程（需用户输入本地密码用于验证）

### 1.2 钱包生成与导入流程

| 步骤 | 生成钱包（Generate）                                   | 导入钱包（Import）                                       |
|------|------------------------------------------------------|--------------------------------------------------------|
| 1    | 用户输入本地密码 → 点击「生成钱包」                         | 用户输入本地密码 → 填写/粘贴助记词 → 点击「导入钱包」            |
| 2    | 客户端对密码进行 SHA-256 哈希，与本地 `userRecord.passwordHash` 比对 | 同左，完全一致的密码校验逻辑                                 |
| 3    | 校验通过 → 调用 `generateWallet()`<br>→ 使用 `crypto.getRandomValues` 生成随机熵<br>→ 同时生成英文 + 中文助记词<br>→ 返回 `WalletResult`（含 `HDNodeWallet`、英文助记词、中文助记词） | 校验通过 → 读取用户填写的助记词（支持 12/24 词、自动识别中英文）<br>→ 调用 `importWallet_en()` 或 `importWallet_zh()`（中文会先转熵再转英文）<br>→ 得到等价的 `HDNodeWallet` 实例 |
| 4    | 调用 `encryptWallet(wallet, password)`<br>→ PBKDF2 派生密钥 + AES-256-GCM 加密整个钱包对象 | 完全相同：`encryptWallet(wallet, password)`（代码复用 100%） |
| 5    | 并行执行：<br>• `countWallet()` 查询已有钱包数量<br>• 加密钱包 | 同左：并行执行 `encryptWallet()` 与 `countWallet()`           |
| 6    | 根据数量决定 `keyPath`：<br>0 个 → `'0'`，其余 → `crypto.randomUUID()` | 完全一致的 keyPath 策略                                 |
| 7    | 并行保存：<br>• `saveWallet(keyPath, encryptedWallet)`<br>• `saveAddress(addressRecord)` | 完全相同的并行写入逻辑（Wallets + Addresses 表）            |
| 8    | 清理内存中的明文助记词与私钥<br>显示「生成成功」<br>提供可切换的英文/中文助记词展示（仅用于备份） | 清理输入框<br>显示「导入成功」<br>提供相同的助记词核对界面（英文 ↔ 中文可切换） |
| 9    | 用户手动备份助记词 → 点击「回到主页」                       | 用户核对助记词无误 → 点击「回到主页」或「前往登录」               |

### 1.3 异常/边界场景

* **密码为空或校验失败**：阻止生成并展示错误提示
* **加密保存失败（IO/Quota）**：回滚或清理已写入的部分数据，提示用户并记录错误日志（可选上报）
* **助记词一致性检查失败**（理论上在 `generateWallet` 已校验）：抛出异常并提示重新生成
* **重复保存/重复按钮点击**：使用 `loading` 状态禁止重复操作

---

## 2. 数据模型（IndexedDB 表结构）

### Wallets 表（`TABLE_WALLETS = 'Wallets'`）

* key: string（当前实现以 `count()` 作为自增 key，再转 String 存储）
* value (`IEncryptedWallet`)（建议示例）

```ts
interface IEncryptedWallet {
  version: number;          // 版本号，方便日后迁移
  cipher: 'AES-GCM';        // 加密算法说明
  iv: string;               // base64/hex
  salt: string;             // 用于 KDF 的 salt
  ciphertext: string;       // 加密后的 payload（可能包含私钥、path、address 等）
  meta?: {
    createdAt: string;      // ISO 时间
    address?: string;
  }
}
```

### Addresses 表（`TABLE_ADDRESSES = 'Addresses'`）

* key: string（同上）
* value (`AddressRecord`)

```ts
interface AddressRecord {
  wallet: {
    type: string;    // 'HDNodeWallet'
    KeyPath: string; // 指向 Wallets 表的 key（String）
  };
  address: string;   // 0x...
}
```

> 注意：当前实现使用 `storage.count()` 作为自增键（key = count），优点简单；缺点是在删除后可能出现 key 缝隙、并发写入风险。可考虑后续改为 UUID 或自增 sequence 管理。

---

## 3. 核心库说明（lib/*.ts）

### `generateWallet.ts`

* 功能：基于 `ethers.Wallet.createRandom()` 生成 HD 钱包，读取英文助记词 entropy，使用 bip39 中文词库生成中文助记词，并做 entropy 一致性校验
* 返回：`WalletResult { wallet: HDNodeWallet, mnemonic_en, mnemonic_zh }`

### `cryptoWallet.ts`

* 功能：钱包加密与解密逻辑（AES-GCM）与密钥派生（应使用 PBKDF2 / scrypt / argon2 类安全 KDF，至少带 salt 与多轮次）。
* 导出：

  * `encryptWallet(wallet: HDNodeWallet, password: string): Promise<IEncryptedWallet>`
  * `decryptWallet(enc: IEncryptedWallet, password: string): Promise<HDNodeWallet>`
* 测试点：解密后钱包地址与加密前一致；篡改 ciphertext/iv/salt 导致解密失败（错误抛出）

### `saveWallet.ts` / `saveAddress.ts`

* 功能：IndexedDB CRUD 封装（`save`, `get`, `delete`, `keys`, `values`, `count` 等）
* 约定：`save*` 返回数字 key（当前实现为 `count()`），`get*` 返回相应记录或 `null/undefined`

---

## 4. 安全设计原则

* **密码永不明文存盘**：仅存储密码的 SHA-256 哈希（用于登录/校验），用于派生密钥的 salt 与派生参数也应存储，但不保存明文密码
* **助记词/私钥最小保留时间**：助记词与私钥仅在内存中存在并在使用后立即覆盖/释放（尽量避免长时驻留）
* **加密存储**：所有敏感 payload 使用用户密码派生密钥（KDF）进行 AES-GCM 加密并存储；加密元信息（salt, iv）可与 ciphertext 一起存
* **防止侧信道泄露**：避免在控制台/日志打印助记词、私钥或完整解密 payload
* **一键删除本地数据**：提供忘记密码/重置功能，彻底删除 Wallets、Addresses、user record
* **版本与迁移**：加密数据带 `version` 字段，便于将来更换加密策略或 KDF 参数时进行迁移
* **最小权限**：模块仅做本地管理，不进行网络同步（除非用户显式导出并上传），避免远端泄露风险










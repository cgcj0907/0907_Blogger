---
title: "EIP-20"
date: 2024-08-20
draft: false
tags: ["Web3", "EIP"]
showToc: true
---

# 摘要
本标准允许在智能合约中实现一个标准化的代币API。它提供了转账代币的基本功能，并允许批准代币以便可以被其他链上第三方花费。

---

# 动机
一个标准化的接口允许以太坊上的任何代币被其他应用程序重复使用：从钱包到去中心化交易所


## 规范

### 代币

#### 方法
>**注意**:
>- 以下规范使用 Solidity 0.4.17（或更高版本）的语法
>- 调用者**必须**处理返回 `false`（`bool success`）的情况。调用者**绝不能**假定 >`false` 永远不会被返回！

1.  **`name`**
    *   **功能**：返回代币的名称 - 例如 `"MyToken"`
    *   **可选性**：此方法可用于提高可用性，但接口和其他合约**绝不能**期望这些值一定存在
    *   `function name() public view returns (string)`

2.  **`symbol`**
    *   **功能**：返回代币的符号。例如 `"HIX"`
    *   **可选性**：此方法可用于提高可用性，但接口和其他合约**绝不能**期望这些值一定存在
    *   `function symbol() public view returns (string)`

3.  **`decimals`**
    *   **功能**：返回代币使用的小数位数 - 例如 `8`，表示将代币数量除以 100000000 以获得其用户表示形式
    *   **可选性**：此方法可用于提高可用性，但接口和其他合约**绝不能**期望这些值一定存在
    *   `function decimals() public view returns (uint8)`

4.  **`totalSupply`**
    *   **功能**：返回代币总供应量
    *   `function totalSupply() public view returns (uint256)`

5.  **`balanceOf`**
    *   **功能**：返回地址 `_owner` 的账户余额
    *   `function balanceOf(address _owner) public view returns (uint256 balance)`

6.  **`transfer`**
    *   **功能**：向地址 `_to` 转移 `_value` 数量的代币，并且**必须**触发 `Transfer` 事件
    *   **要求**：如果消息调用者的账户余额没有足够的代币可以花费，该函数**应该**抛出异常
    *   **注意**：零值的转移**必须**被视为正常转移并触发 `Transfer` 事件
    *   `function transfer(address _to, uint256 _value) public returns (bool success)`

7.  **`transferFrom`**
    *   **功能**：从地址 `_from` 向地址 `_to` 转移 `_value` 数量的代币，并且**必须**触发 `Transfer` 事件
    *   **用途**：`transferFrom` 方法用于提款工作流程，允许合约代表你转移代币。例如，这可用于允许合约代表你转移代币和/或以子货币收取费用
    *   **要求**：除非 `_from` 账户已通过某种机制有意授权了消息的发送者，否则该函数**应该**抛出异常
    *   **注意**：零值的转移**必须**被视为正常转移并触发 `Transfer` 事件
    *   `function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)`

8.  **`approve`**
    *   **功能**：允许 `_spender` 从你的账户中多次提取，最高达 `_value` 金额。如果再次调用此函数，它将用 `_value` 覆盖当前的 `allownce`
    *   **重要安全说明**：为了防止像[这里](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit)描述和[这里](https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729)讨论的攻击向量，**客户端应确保创建用户界面的方式是，在为同一支出者设置另一个值之前，先将 `allowance` 设置为0。** 但是，合约本身不应强制执行此操作，以便与之前部署的合约保持向后兼容性
    *   `function approve(address _spender, uint256 _value) public returns (bool success)`

9.  **`allowance`**
    *   **功能**：返回 `_spender` 仍然被允许从 `_owner` 提取的金额
    *   `function allowance(address _owner, address _spender) public view returns (uint256 remaining)`

#### 事件

1.  **`Transfer`**
    *   **功能**：当代币被转移时**必须**触发，包括零值转移
    *   **铸造新代币**：创建新代币的代币合约在创建代币时，**应该**触发一个 `Transfer` 事件，并将 `_from` 地址设置为 `0x0`
    *   `event Transfer(address indexed _from, address indexed _to, uint256 _value)`

2.  **`Approval`**
    *   **功能**：在任何对 `approve(address _spender, uint256 _value)` 的成功调用上**必须**触发
    *   `event Approval(address indexed _owner, address indexed _spender, uint256 _value)`

---

### 实现
以太坊网络上已经部署了大量符合 ERC20 标准的代币。不同的团队已经编写了具有不同权衡（从节省 Gas 到提高安全性）的实现。

**示例实现可在以下位置找到**：
*   [OpenZeppelin 实现](https://github.com/OpenZeppelin/openzeppelin-contracts)
*   [ConsenSys 实现](https://github.com/ConsenSys/Tokens)

---


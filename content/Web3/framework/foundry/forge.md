---
title: "foundry Forge 使用指南"
date: 2025-08-20
draft: false
tags: ["Web3", "framework", "foundry"]
showToc: true
---

# 简介

Forge 是 Foundry 的核心组件，用于编写、运行和调试 Solidity 智能合约测试，支持单元测试、模糊测试（fuzz testing）和不变量测试（invariant testing）。它允许开发者在 Solidity 中直接编写测试，而无需切换到其他语言。Forge 通过 `forge test` 命令运行测试，并提供丰富的作弊码（cheatcodes）来模拟各种场景


## 测试类型

### Stateless Test

无状态测试（Stateless Test）是指每个测试函数独立运行，状态在每次调用后重置。这包括标准的单元测试和无状态模糊测试。在无状态测试中，Forge 会为每个测试函数创建一个干净的环境，确保测试之间互不干扰。这适合测试单一函数的行为或特定输入输出


### Stateful Test

有状态测试（Stateful Test）允许测试在多个函数调用序列中保持状态，通常用于不变量测试（Invariant Testing）。在这种测试中，Forge 会生成随机调用序列，并在每个调用后检查不变量是否成立。这有助于发现复杂协议中的逻辑错误，尤其是在多步交互场景下


## forge 基础测试

### 文件目录与命名

Foundry 项目采用标准的目录结构，便于管理和测试：

- **src/**：存放核心智能合约代码，例如 `Contract.sol`
- **test/**：存放测试合约，通常以 `.t.sol` 结尾，例如 `Contract.t.sol`。测试文件放在这里以便 Forge 自动识别
- **script/**：存放部署脚本，例如 `Deploy.s.sol`
- **lib/**：存放依赖库，如 forge-std 或 OpenZeppelin
- **foundry.toml**：配置文件，用于设置测试参数、依赖等

命名规范：
- 测试合约文件名以 `.t.sol` 结尾
- 源合约文件名以 `.sol` 结尾
- 避免使用特殊字符，确保文件名描述性强


### 合约代码框架

Forge 测试合约必须继承自 `forge-std/Test.sol` 中的 `Test` 合约，这提供了断言、日志和作弊码功能。基本框架如下：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";

contract MyContractTest is Test {
    // 待测试合约实例
    MyContract public myContract;

    // setUp 函数：在每个测试前运行，初始化状态
    function setUp() public {
        myContract = new MyContract();
    }

    // 测试函数：以 test 开头
    function test_Example() public {
        // 测试逻辑
        assertTrue(true);
    }
}
```

- **setUp()**：可选，用于初始化合约或状态。
- 测试函数：必须以 `test` 开头，public 或 external
- 断言：使用 `assertEq(a, b)`、`assertTrue(condition)` 等
- 作弊码：如 `vm.prank(address)` 模拟调用者

## forge 进阶测试

### Fuzz Testing

模糊测试（Fuzz Testing）是一种  property based 测试，Forge 会生成随机输入来验证合约的通用行为。测试函数需至少有一个参数（如 `uint256 amount`），Forge 会自动将其视为 fuzz 测试，默认运行 256 次

- **假设（Assumptions）**：使用 `vm.assume(condition)` 排除无效输入，例如 `vm.assume(amount > 0)`
- **限制输入**：使用较小类型如 `uint96` 避免溢出
- **fixture**：使用数组或函数提供固定值，例如 `uint32[] public fixtureAmounts = [1, 5, 555];`。

示例：

```solidity
function testFuzz_Withdraw(uint96 amount) public {
    vm.assume(amount > 0 && amount <= address(this).balance);
    // 测试逻辑
    assertEq(beforeBalance - afterBalance, amount);
}
```



## Invariant Testing

不变量测试（Invariant Testing）是一种有状态模糊测试，Forge 会对预定义合约生成随机函数调用序列，并在每个调用后检查不变量（始终成立的条件）。适合测试协议的整体一致性，如 ERC-20 的 totalSupply == sum(balances)

- **设置**：测试函数以 `invariant` 开头，如 `invariant_TotalSupply()`。
- **目标**：指定 targetContracts、targetSelectors 等控制 fuzz 范围
- **Handler**：使用中间合约管理状态，确保有效调用
- **配置**：runs、depth、fail_on_revert 等。
- **Ghost Variables**：跟踪辅助状态，如 sumDeposits

示例：

```solidity
contract InvariantTest is StdInvariant, Test {
    MyProtocol public protocol;

    function setUp() public {
        protocol = new MyProtocol();
        targetContract(address(protocol));
    }

    function invariant_BalanceSum() public {
        assertEq(protocol.totalSupply(), protocol.balanceOf(user1) + protocol.balanceOf(user2));
    }
}
```

### Open Testing

### Handler-Based Testing


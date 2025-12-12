---
title: "漏洞等级划分标准"
date: 2025-08-16
draft: false
tags: ["Web3", "security"]
showToc: true
---
# 简介

在识别代码库中的漏洞时，评估其潜在影响与风险至关重要。漏洞（finding）通常分为三类：

* **High（高危）**
* **Medium（中危）**
* **Low（低危）**



## 如何评估漏洞严重性

漏洞的严重性可被归类为 **High（高）**、**Medium（中）** 或 **Low（低）**，其判定基于多个因素：

1. **对协议的影响（Impact）**：如果漏洞被利用，会造成多严重的损害？
2. **被利用的可能性（Likelihood）**：攻击者利用该漏洞的概率有多大？
3. **评审/协议主观性程度（Degree of judge/protocol subjectivity）**

下面用矩阵给出一个结构化参考：
| 可能性 \ 影响       | 高 (High)             | 中 (Medium)           | 低 (Low)             |
| -------------- | -------------------- | -------------------- | ------------------- |
| **高 (High)**   | 🔴 **High**          | 🟠 **High / Medium** | 🟡 **Medium**       |
| **中 (Medium)** | 🟠 **High / Medium** | 🟡 **Medium**        | 🟢 **Medium / Low** |
| **低 (Low)**    | 🟡 **Medium**        | 🟢 **Medium / Low**  | 🔵 **Low**          |


**分类的主观性**

虽然“影响 × 可能性”矩阵提供了结构化方法，但分类仍然带有一定主观性，判断者（judge）的裁量权在最终分类中起关键作用

如果被审计的协议有明确的判定标准，则应以协议规定为判定基准


### 如何评估漏洞的影响（Impact）

Impact 指漏洞被利用后对用户或协议造成的潜在损害或后果

* **High（高影响）**：

  * 资金直接或几乎直接处于风险中。
  * 协议功能或可用性发生严重中断。
* **Medium（中影响）**：

  * 资金间接处于风险中。
  * 对协议功能或可用性造成一定程度的中断或异常。
* **Low（低影响）**：

  * 资金不处于风险中。
  * 可能存在功能不正确、状态处理不当等问题，但不会导致直接资金损失。

### 如何评估漏洞被利用的**可能性（Likelihood）**

Likelihood 表示由于漏洞而导致影响发生的概率

<details>
<summary>High（高可能性）</summary>

非常可能发生。例如：攻击者可以直接调用某个函数并提取资金

</details>

<details>
<summary>Medium（中可能性）</summary>

可能在特定条件下发生。例如：平台接入了某个特殊的 ERC20 代币，该代币具有非标准行为

</details>

<details>
<summary>Low（低可能性）</summary>

不太可能发生。例如：某变量被设置为在特定区块号上才有意义且难以改变

</details>


**注意**
有些情况下可能性被判定为“计算上不可行”（computationally infeasible）。例如，“攻击者猜出用户私钥”。在此类情形下，提交者必须证明其发现是**计算上可行**的


## 高、中、低 级别漏洞示例

### High（高风险）示例

**特性：**

* 对资金或协议主要功能有直接影响。
* 攻击路径直接明了。
* 漏洞易被利用，可能造成重大损失。

**示例：**
[查看高风险示例（Detailed High Severity Finding）](https://solodit.xyz/issues/boostsetlockstatus-should-update-the-callers-rewards-first-hans-none-meta-markdown_)

更多高风险示例，请访问 [Solodit](https://solodit.xyz/)。

### Medium（中风险）示例

**特性：**

* 对资金或协议功能有间接影响。
* 攻击路径不直接，需要满足特定条件。
* 虽然可能造成损害，但利用难度较高。

**示例：**
[查看中风险示例（Detailed Medium Severity Finding）](https://solodit.xyz/issues/the-off-chain-mechanism-must-be-ensured-to-work-in-a-correct-order-strictly-cyfrin-none-cyfrin-stake-link-markdown)

更多中风险示例，请访问 [Solodit](https://solodit.xyz/)。

### Low（低风险）示例

**特性：**

* 对资金或协议主要功能影响极小。
* 漏洞不会导致现实世界的可观损失。
* 利用路径不存在或极其不可能。

**示例：**
[查看低风险示例（Detailed Low Severity Finding）](https://solodit.xyz/issues/l-06-erc1155action-returns-false-on-supportsinterface-with-the-real-erc1155-interface-code4rena-notional-notional-contest-git)

更多示例，请访问 [Solodit](https://solodit.xyz/)



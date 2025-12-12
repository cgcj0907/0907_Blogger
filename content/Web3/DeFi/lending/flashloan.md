
---
title: "Flash Loan 闪电贷"
date: 2025-08-20
draft: false
tags: ["Web3", "DeFi"]
showToc: true
---
# 简介

**闪电贷（Flash Loan）** 是一种在单笔区块链交易内借入资产、使用资产并在同一交易内归还借款（含手续费）的借贷模式。核心特点是 **“原子性”**：借款、使用和还款必须在同一笔交易内完成，否则交易会回滚，借款不会生效。

这种机制使得借款人在不提供抵押的情况下临时动用大量资产，用于套利、清算、头寸迁移等操作；同时也被用于发动复杂的攻击（如操纵价格、闪电清算等）。

---

## 原理与执行流程

典型单笔闪电贷交易包含三步（原子性）：

1. **借款（Borrow）**
   在交易开始阶段通过闪电贷提供方（Lending Pool/Router）请求借入若干资产。

2. **使用（Use）**
   在同一交易中，用借入资金执行任意链上操作，例如在不同 DEX 之间套利、进行清算、调整抵押头寸、执行跨协议操作等。

3. **还款（Repay）**
   在交易结束前，将借入金额加上借贷方要求的手续费一并偿还给闪电贷提供方。如果无法偿还（余额不足或逻辑失败），整笔交易 revert，等同于“未发生”。

关键点：闪电贷的安全性与“回滚语义”绑定；若中间步骤出错，借贷不会被实际放出和使用。

---

## 常见闪电贷提供方

* Aave（flashLoan / flashLoanSimple 接口）
* DyDx（Solo or v3）
* Uniswap（flash swap：允许先输出资产，再在同一交易内履约）
* Balancer / Curve / 其它支持 flash 的池子
  （实现细节与接口在不同协议间不同）

---

## 典型用途（Use cases）

* **跨 DEX 套利**：利用不同交易所价格差套利，套利收益用于偿还贷款并留盈利
* **清算（Liquidation）**：借资产替别人清算欠债（避免自己需提前持有资产）
* **头寸迁移 / 债务重组**：一笔交易内把抵押品替换、债务迁移到另一个协议或仓位
* **杠杆构建 / 扩展**：临时放大仓位用于策略
* **原子化多步操作**：把多个必须同时完成的步骤整合为一笔交易，避免中间状态风险

---


## 代码示例
```solidity
// SPDX-License-Identifier: AGPL-3.0-only
pragma solidity 0.8.20;

import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import { AssetToken } from "./AssetToken.sol";
import { IERC20, IERC20Metadata } from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";
import { Oracle } from "./Oracle.sol";

interface IFlashLoanReceiver {
    function executeOperation(address token, uint256 amount, uint256 fee, address initiator, bytes calldata params) external;
}

contract ThunderLoan is Ownable, Oracle {
    using SafeERC20 for IERC20;
    
    /*//////////////////////////////////////////////////////////////
                            STATE VARIABLES
    //////////////////////////////////////////////////////////////*/
    mapping(IERC20 => AssetToken) public s_tokenToAssetToken;
    mapping(IERC20 => bool) private s_currentlyFlashLoaning;
    
    uint256 private constant FEE_PRECISION = 1e18;
    uint256 private s_flashLoanFee = 3e15; // 0.3%
    
    /*//////////////////////////////////////////////////////////////
                                 EVENTS
    //////////////////////////////////////////////////////////////*/
    event Deposit(address indexed account, IERC20 indexed token, uint256 amount);
    event Withdraw(address indexed account, IERC20 indexed token, uint256 amount);
    event FlashLoan(address indexed receiver, IERC20 indexed token, uint256 amount, uint256 fee);
    event TokenAllowed(IERC20 indexed token, bool allowed);
    
    /*//////////////////////////////////////////////////////////////
                               MODIFIERS
    //////////////////////////////////////////////////////////////*/
    modifier notZero(uint256 amount) {
        require(amount > 0, "Amount cannot be zero");
        _;
    }
    
    modifier allowedToken(IERC20 token) {
        require(address(s_tokenToAssetToken[token]) != address(0), "Token not allowed");
        _;
    }
    
    /*//////////////////////////////////////////////////////////////
                           EXTERNAL FUNCTIONS
    //////////////////////////////////////////////////////////////*/
    constructor(address oracle) Oracle(oracle) Ownable(msg.sender) {}
    
    function deposit(IERC20 token, uint256 amount) external notZero(amount) allowedToken(token) {
        AssetToken assetToken = s_tokenToAssetToken[token];
        
        // 计算应铸造的资产代币数量
        uint256 mintAmount = (amount * FEE_PRECISION) / assetToken.getExchangeRate();
        
        // 铸造资产代币给用户
        assetToken.mint(msg.sender, mintAmount);
        
        // 更新汇率（包含手续费）
        assetToken.updateExchangeRate(calculateFee(token, amount));
        
        // 将底层代币转移到资产代币合约
        token.safeTransferFrom(msg.sender, address(assetToken), amount);
        
        emit Deposit(msg.sender, token, amount);
    }
    
    function withdraw(IERC20 token, uint256 assetAmount) external notZero(assetAmount) allowedToken(token) {
        AssetToken assetToken = s_tokenToAssetToken[token];
        
        // 计算可取回的底层代币数量
        uint256 underlyingAmount = (assetAmount * assetToken.getExchangeRate()) / FEE_PRECISION;
        
        // 销毁用户的资产代币
        assetToken.burn(msg.sender, assetAmount);
        
        // 将底层代币转给用户
        assetToken.transferUnderlyingTo(msg.sender, underlyingAmount);
        
        emit Withdraw(msg.sender, token, underlyingAmount);
    }
    
    function flashLoan(
        address receiver,
        IERC20 token,
        uint256 amount,
        bytes calldata params
    ) external notZero(amount) allowedToken(token) {
        AssetToken assetToken = s_tokenToAssetToken[token];
        
        // 检查合约余额是否足够
        uint256 poolBalance = token.balanceOf(address(assetToken));
        require(amount <= poolBalance, "Insufficient pool balance");
        
        // 检查接收者是合约
        require(receiver.code.length > 0, "Receiver must be contract");
        
        // 计算手续费
        uint256 fee = calculateFee(token, amount);
        
        // 更新汇率
        assetToken.updateExchangeRate(fee);
        
        // 标记为正在闪电贷
        s_currentlyFlashLoaning[token] = true;
        
        // 将资金转给接收者
        assetToken.transferUnderlyingTo(receiver, amount);
        
        // 调用接收者的回调函数
        IFlashLoanReceiver(receiver).executeOperation(
            address(token),
            amount,
            fee,
            msg.sender,
            params
        );
        
        // 验证还款
        uint256 newBalance = token.balanceOf(address(assetToken));
        require(newBalance >= poolBalance + fee, "Flash loan not repaid");
        
        // 重置闪电贷状态
        s_currentlyFlashLoaning[token] = false;
        
        emit FlashLoan(receiver, token, amount, fee);
    }
    
    function repay(IERC20 token, uint256 amount) external {
        require(s_currentlyFlashLoaning[token], "Not in flash loan");
        token.safeTransferFrom(msg.sender, address(s_tokenToAssetToken[token]), amount);
    }
    
    /*//////////////////////////////////////////////////////////////
                            ADMIN FUNCTIONS
    //////////////////////////////////////////////////////////////*/
    function setAllowedToken(IERC20 token, bool allowed) external onlyOwner {
        if (allowed) {
            require(address(s_tokenToAssetToken[token]) == address(0), "Already allowed");
            
            string memory name = string.concat("ThunderLoan ", IERC20Metadata(address(token)).name());
            string memory symbol = string.concat("tl", IERC20Metadata(address(token)).symbol());
            
            AssetToken assetToken = new AssetToken(address(this), token, name, symbol);
            s_tokenToAssetToken[token] = assetToken;
        } else {
            delete s_tokenToAssetToken[token];
        }
        
        emit TokenAllowed(token, allowed);
    }
    
    function setFlashLoanFee(uint256 newFee) external onlyOwner {
        require(newFee <= FEE_PRECISION, "Fee too high");
        s_flashLoanFee = newFee;
    }
    
    /*//////////////////////////////////////////////////////////////
                            VIEW FUNCTIONS
    //////////////////////////////////////////////////////////////*/
    function calculateFee(IERC20 token, uint256 amount) public view returns (uint256) {
        uint256 tokenValue = (amount * getPriceInWeth(address(token))) / FEE_PRECISION;
        return (tokenValue * s_flashLoanFee) / FEE_PRECISION;
    }
    
    function isAllowedToken(IERC20 token) public view returns (bool) {
        return address(s_tokenToAssetToken[token]) != address(0);
    }
    
    function getFlashLoanFee() external view returns (uint256) {
        return s_flashLoanFee;
    }
}
```

## 经济可行性与可行性检查

在执行闪电贷策略前需判断经济可行性，常见校验条目：

* **预估毛收益（grossProfit）**：例如套利两侧成交价差带来的利润（减去滑点与手续费）。
* **借贷费用（fee）**：闪电贷提供方收取的费率（必须包含在还款中）。
* **交易费用（gas）**：执行整笔复杂交易的 gas 费用（尤其在以太坊主网或 L2）。
* **滑点与深度**：大额操纵会导致滑点，稀薄池子会显著降低利润。
* **回滚风险**：若中间任一步失败，整笔交易回滚，需评估失败成本（主要为失败的 gas 费）。

简单可行性条件：`estimatedProfitAfterSlippage - fees - gas > 0`。在脚本中模拟不同滑点与 gas 情形，保守估算

---







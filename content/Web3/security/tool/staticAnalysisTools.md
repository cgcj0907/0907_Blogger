---
title: "静态分析工具"
date: 2025-08-16
draft: false
tags: ["Web3", "security", "Toolkits"]
showToc: true
---

# 简介

静态分析工具是智能合约安全审计的重要辅助手段，能够在代码执行前自动检测潜在的安全漏洞、编码问题和最佳实践违规。这些工具通过分析源代码或字节码的模式来识别风险，极大提高了审计效率和漏洞发现率。

# 主要静态分析工具对比

| 工具名称 | 语言支持 | 分析类型 | 特点 | 适用场景 |
|---------|---------|---------|------|----------|
| **Slither** | Solidity | 源代码分析 | 快速、检测种类多、可定制规则 | 常规安全审计、开发中检查 |
| **Mythril** | EVM字节码 | 符号执行 | 深度分析、路径覆盖广 | 复杂逻辑漏洞、重入攻击检测 |
| **Aderyn** | Solidity | 源代码分析 | 专注于Rust实现、速度快 | Rust开发者、性能敏感场景 |
| **Semgrep** | 多语言 | 模式匹配 | 简单规则、学习曲线低 | 团队编码规范检查 |

# [Slither](https://github.com/crytic/slither)

## 概述
Slither 是由 Trail of Bits 开发的 Solidity 静态分析框架，速度快、检测能力强，支持自定义检测规则。

## 安装
```console
python3 -m pip install slither-analyzer
```

## 升级
```console
python3 -m pip install --upgrade slither-analyzer
```

## 基本使用
### 扫描当前目录
```console
slither .
```
### 扫描特定文件
```console
slither tests/uninitialized.sol
```

## 检测器选择
Slither 默认会运行 所有内置检测器
### 指定检测器
```console
slither file.sol --detect arbitrary-send,pragma
```
### 排除指定检测器
```console
slither file.sol --exclude naming-convention,unused-state,suicidal
```
## 排除全部 informational / low 级别的检测器
```console
slither file.sol --exclude-informational
slither file.sol --exclude-low
```
## 查看所有可用检测器
```console
slither --list-detectors
```
## 打印器
默认情况下，所有 printers 都不会运行
### 打印器选择
```console
slither file.sol --print inheritance-graph
```
### 列出所有可用打印器
```console
slither --list-printers

```
## 路劲过滤
--filter-paths 用于排除 只与某些路径相关的结果

可使用路径、文件名、字符串匹配或正则表达式

示例：
1. 排除所有来自 openzeppelin 的结果
```console
slither . --filter-paths "openzepellin"
```
2. 排除两个特定文件的所有结果
```console
slither . --filter-paths "Migrations.sol|ConvertLib.sol"
```
## 分诊模式
Slither 提供两种方式排除结果：

1. 在代码上添加注释（禁用某个检测器）
▪ 禁用下一行
```solidity
// slither-disable-next-line DETECTOR_NAME
```
▪ 禁用一段代码
```solidity
// slither-disable-start detector-name
    ...（被忽略的代码）...
// slither-disable-end detector-name
```

2. 声明某变量是非可重入（让 Slither 不提示可重入相关问题）
```solidity
@custom:security non-reentrant
uint256 target;
```
## 交互式分诊模式
* 使用 --triage-mode：
```console
slither . --triage-mode
````

* 运行后，每条发现都会询问：
```console
Results to hide during next runs: "0,1,..." or "All" (enter to not hide results):
```
```
输入某些编号 → 将在下一次运行中隐藏
输入“All” → 隐藏本轮所有结果
直接回车 → 不隐藏

Slither 会把隐藏规则保存在：
`slither.db.json`

示例: 
```console
slither . --triage-mode
[...]
0: C.destination (test.sol#3) is never initialized. It is used in:
	- f (test.sol#5-7)
Reference: https://github.com/trailofbits/slither/wiki/Vulnerabilities-Description#uninitialized-state-variables
Results to hide during next runs: "0,1,..." or "All" (enter to not hide results):  0
[...]
```
* 显示隐藏结果
删除该文件即可：
rm slither.db.json

## 配置文件
默认情况下, 配置文件为 `slither.config.json`
可通过以下命令更改:
```console
slither . --config-file file.config.json
```
### 常用配置项
```json
{
    "detectors_to_exclude": "conformance-to-solidity-naming-conventions,incorrect-versions-of-solidity", //example
    "exclude_informational": False,
    "exclude_low": False,
    "exclude_medium": False,
    "exclude_high": False,
    "disable_color": False,
    "filter_paths": "(mocks/|test/|script/|upgradedProtocol/)", // example
    "legacy_ast": False,
    "exclude_dependencies": True
}
```

### 详细配置选项
```json
{
    "detectors_to_run": "all",
    "printers_to_run": None,
    "detectors_to_exclude": None,
    "detectors_to_include": None,
    "exclude_dependencies": False,
    "exclude_informational": False,
    "exclude_optimization": False,
    "exclude_low": False,
    "exclude_medium": False,
    "exclude_high": False,
    "fail_on": FailOnLevel.PEDANTIC,
    "json": None,
    "sarif": None,
    "disable_color": False,
    "filter_paths": None,
    "include_paths": None,
    "generate_patches": False,
    "skip_assembly": False,
    "legacy_ast": False,
    "zip": None,
    "zip_type": "lzma",
    "show_ignored_findings": False,
    "sarif_input": "export.sarif",
    "sarif_triage": "export.sarif.sarifexplorer",
    "triage_database": "slither.db.json",
    # codex
    "codex": False,
    "codex_contracts": "all",
    "codex_model": "text-davinci-003",
    "codex_temperature": 0,
    "codex_max_tokens": 300,
    "codex_log": False,
}
```


## 其他重要工具

### Mythril
```bash
# 安装
pip install mythril

# 基本使用
myth analyze contract.sol
myth analyze -x contract.sol  # 排除检测器

# 符号执行选项
myth analyze contract.sol --execution-timeout 600
myth analyze contract.sol --parallel-solving
```

### Semgrep for Solidity
```bash
# 安装
pip install semgrep

# 使用官方规则
semgrep --config p/solidity contract.sol

# 自定义规则
# 创建 rule.yaml:
# rules:
#   - id: unchecked-transfer
#     pattern: |
#       $TOKEN.transfer(...);
#     message: "Unchecked transfer() call"
```

### Foundry 内置检查
```bash
# 使用 Forge 的检查命令
forge inspect contract.sol --check

# 查看 Gas 报告
forge test --gas-report

# 查看存储布局
forge inspect contract.sol --storage-layout
```

### Echidna 属性测试
```bash
# 安装
curl https://raw.githubusercontent.com/crytic/echidna/master/install.sh | bash

# 创建属性测试合约
# 运行测试
echidna-test contract.sol --config config.yaml
```

## 综合审计流程示例

### 步骤1：初步扫描
```bash
# 使用多个工具进行初步检查
slither . --exclude-informational --json slither-report.json
aderyn . --format json --output aderyn-report.json
myth analyze . --max-depth 12 --execution-timeout 900
```

### 步骤2：结果整合
```python
# 整合多个工具结果的脚本示例
import json

def merge_reports():
    reports = []
    
    with open('slither-report.json', 'r') as f:
        reports.append(('Slither', json.load(f)))
    
    with open('aderyn-report.json', 'r') as f:
        reports.append(('Aderyn', json.load(f)))
    
    # 去重、按严重性排序、生成统一报告
    # ...
```

### 步骤3：人工复核
```markdown
# 审计检查清单

## 高优先级问题
- [ ] 重入漏洞 (检查 slither, mythril 报告)
- [ ] 权限控制缺失 (检查 aderyn 报告)
- [ ] 算术溢出 (检查所有工具的数学运算报告)
- [ ] 未检查的低级调用 (检查 send/call/delegatecall)

## 中优先级问题
- [ ] Gas 优化建议
- [ ] 事件参数缺失
- [ ] 错误处理不完整
- [ ] 时间戳依赖

## 低优先级问题
- [ ] 命名规范问题
- [ ] 注释不完整
- [ ] 代码复杂度过高
```

## 最佳实践

### 1. 开发阶段集成
```yaml
# pre-commit 配置示例 (.pre-commit-config.yaml)
repos:
  - repo: https://github.com/crytic/slither
    rev: v0.9.5
    hooks:
      - id: slither
  
  - repo: local
    hooks:
      - id: aderyn
        name: Aderyn
        entry: aderyn
        args: [".", "--format", "compact"]
        language: system
        pass_filenames: false
```

### 2. 团队规范配置
```yaml
# 团队共享的检测配置 (team-rules.yaml)
detectors:
  # 必须启用的安全检测
  mandatory:
    - reentrancy-eth
    - unchecked-lowlevel
    - suicidal
    - delegatecall-loop
  
  # 建议启用的质量检测
  recommended:
    - naming-convention
    - external-function
    - constable-states
  
  # 团队自定义规则
  custom:
    - id: no-hardcoded-address
      pattern: "0x[0-9a-fA-F]{40}"
      message: "Avoid hardcoded addresses, use constants"
```

### 3. 持续监控
```bash
# 定期扫描脚本
#!/bin/bash
# daily-scan.sh

DATE=$(date +%Y%m%d)
REPORT_DIR="reports/$DATE"

mkdir -p $REPORT_DIR

# 运行扫描
slither . --json $REPORT_DIR/slither.json
aderyn . --format markdown --output $REPORT_DIR/aderyn.md
myth analyze . --execution-timeout 1200 > $REPORT_DIR/mythril.txt

# 比较与前一天的结果
python scripts/compare_reports.py $REPORT_DIR "reports/$(date -d yesterday +%Y%m%d)"
```

## 局限性说明

### 静态分析工具的局限性

1. **误报和漏报**
   - 无法理解业务逻辑的特定上下文
   - 可能报告无害的模式为漏洞

2. **分析深度限制**
   - 符号执行工具可能无法覆盖所有代码路径
   - 复杂合约的分析时间可能很长

3. **环境依赖**
   - 需要完整的项目结构才能进行准确分析
   - 依赖项缺失可能导致分析不完整

4. **新漏洞模式**
   - 只能检测已知模式的漏洞
   - 新型攻击手段需要手动编写检测规则

### 建议使用策略
- 将静态分析作为**辅助工具**而非唯一手段
- **人工审计**仍然必不可少
- 结合**动态测试**和**形式验证**
- 建立**多层防御**体系


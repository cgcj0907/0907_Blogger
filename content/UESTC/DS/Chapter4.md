---
title: "第 4 章 串"
date: 2026-04-20
draft: false
tags: ["UESTC", "数据结构"]
ShowToc: true
---

## 串的存储结构

<style>
  /* Chapter4 串结构示意图样式（内嵌 HTML 使用） */
  .ds-diagram { margin: 10px 0 18px; font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; }
  .ds-row { display: flex; align-items: center; gap: 6px; flex-wrap: wrap; }
  .ds-label { font-size: 12px; color: #555; margin-bottom: 6px; }
  .ds-box { min-width: 34px; height: 30px; border: 1px solid #333; display:flex; align-items:center; justify-content:center; font-size: 12px; background: #fff; border-radius: 6px; box-shadow: 0 1px 0 rgba(0,0,0,.06); }
  .ds-box.dim { color: #999; border-style: dashed; }
  .ds-box.head { background: #f4f7ff; }
  .ds-box.ptr  { background: #fff7e6; min-width: 44px; }
  .ds-ellipsis { padding: 0 4px; color: #666; }
  .ds-arrow { font-size: 14px; color: #333; padding: 0 2px; }
  .ds-note { font-size: 12px; color: #666; margin-top: 6px; line-height: 1.4; }
  .ds-sub { font-size: 11px; color: #666; }

  /* 节点：data | next（链串） */
  .ds-node {
    display: inline-flex;
    align-items: stretch;
    height: 30px;
    border: 1px solid #333;
    border-radius: 6px;
    overflow: hidden;
    background: #fff;
    box-shadow: 0 1px 0 rgba(0,0,0,.06);
  }
  .ds-node .data {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0 12px;
    font-size: 12px;
    min-width: 34px;
  }
  .ds-node .next {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0 12px;
    font-size: 11px;
    min-width: 54px;
    border-left: 1px solid #333;
    background: #fff7e6;
    color: #333;
  }

  /* 块结点：block(k个字符) | next（块链串） */
  .ds-bnode {
    display: inline-flex;
    align-items: stretch;
    height: 30px;
    border: 1px solid #333;
    border-radius: 6px;
    overflow: hidden;
    background: #fff;
    box-shadow: 0 1px 0 rgba(0,0,0,.06);
  }
  .ds-bnode .block {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0 10px;
    font-size: 12px;
    min-width: 96px;
    letter-spacing: 2px;
  }
  .ds-bnode .next {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0 12px;
    font-size: 11px;
    min-width: 54px;
    border-left: 1px solid #333;
    background: #fff7e6;
    color: #333;
  }
</style>

### 定长顺序存储（静态数组）

<div class="ds-diagram">
  <div class="ds-label">示意：定长顺序存储（MaxSize 固定，未用单元可能浪费）</div>
  <div class="ds-row" aria-label="fixed-array-string">
    <div class="ds-box">a</div>
    <div class="ds-box">b</div>
    <div class="ds-box">a</div>
    <div class="ds-box">c</div>
    <div class="ds-box dim">∅</div>
    <div class="ds-box dim">∅</div>
    <div class="ds-box dim">∅</div>
    <span class="ds-ellipsis">...</span>
    <div class="ds-box dim">∅</div>
  </div>
  <div class="ds-note">
    <span class="ds-sub">length=4</span>，其余为预留空间（MaxSize-length）
  </div>
</div>

- **存储方式**：用连续存储单元（数组）依次存放字符
- **特点**：
  - 随机访问方便：可 `O(1)` 取第 `i` 个字符
  - 需要预先给出 `MaxSize`，可能出现空间浪费或溢出
- **常见实现**：额外记录 `length`（串长），便于快速求长度

### 堆分配顺序存储（动态数组）

- **存储方式**：按需动态申请连续空间（如 `malloc/new`），串增长时可扩容
- **特点**：
  - 空间利用率更高，适合长度变化较大场景
  - 扩容可能带来拷贝开销

<div class="ds-diagram">
  <div class="ds-label">示意：堆分配顺序存储（capacity 可扩容）</div>
  <div class="ds-row" aria-label="heap-array-string">
    <div class="ds-box head">base</div>
    <span class="ds-arrow">→</span>
    <div class="ds-box">a</div>
    <div class="ds-box">b</div>
    <div class="ds-box">a</div>
    <div class="ds-box">c</div>
    <div class="ds-box dim">∅</div>
    <div class="ds-box dim">∅</div>
  </div>
  <div class="ds-note">
    <span class="ds-sub">length=4</span>，<span class="ds-sub">capacity=6</span>（增长时可申请更大块并拷贝）
  </div>
</div>

### 链式存储（链串）

<div class="ds-diagram">
  <div class="ds-label">示意：链串（每个结点 1 个字符 + 1 个指针域）</div>
  <div class="ds-row" aria-label="linked-string">
    <div class="ds-box head">head</div>
    <span class="ds-arrow">→</span>
    <div class="ds-node"><span class="data">a</span><span class="next">next</span></div>
    <span class="ds-arrow">→</span>
    <div class="ds-node"><span class="data">b</span><span class="next">next</span></div>
    <span class="ds-arrow">→</span>
    <div class="ds-node"><span class="data">a</span><span class="next">next</span></div>
    <span class="ds-arrow">→</span>
    <div class="ds-node"><span class="data">c</span><span class="next">NULL</span></div>
  </div>
  <div class="ds-note">优点：插入/删除不必整体移动；缺点：指针域带来额外开销、随机访问差</div>
</div>

- **存储方式**：用链表结点存放字符并用指针连接
- **特点**：
  - 插入/删除在局部位置更灵活（不必整体移动）
  - 存储密度低（指针域开销），随机访问不便（取第 `i` 个字符需 `O(i)` 遍历）

### 块链存储（块链串）

<div class="ds-diagram">
  <div class="ds-label">示意：块链串（每结点存 k 个字符，末尾可能有空位）</div>
  <div class="ds-row" aria-label="block-linked-string">
    <div class="ds-box head">head</div>
    <span class="ds-arrow">→</span>
    <div class="ds-bnode"><span class="block">a&nbsp;b&nbsp;a</span><span class="next">next</span></div>
    <span class="ds-arrow">→</span>
    <div class="ds-bnode"><span class="block">c&nbsp;∅&nbsp;∅</span><span class="next">NULL</span></div>
  </div>
  <div class="ds-note">
    例：块大小 <span class="ds-sub">k=3</span>。相比链串减少指针数；相比顺序串允许分块扩展
  </div>
</div>

- **核心思想**：一个结点存放多个字符（一个“块”），再用指针把块串起来
- **特点**：
  - 相比链串：减少指针数量，提高存储密度
  - 相比顺序存储：仍保留一定插入/删除灵活性
- **结点结构要点**：
  - 结点内字符数组大小 `k`（块大小）是性能折中：`k` 越大越接近顺序存储，`k` 越小越接近链式存储

## 模式匹配

### 朴素匹配（BF / 暴力匹配）

- **思想**：从主串某一位置开始，逐字符与模式串比较；一旦失配，主串起始位置后移一位，模式串从头再比
- 设主串长度 `n`，模式串长度 `m`：
  - **最好情况**：一次就匹配成功，约 `O(m)`
  - **最坏情况**：大量“前缀相同、末尾失配”，时间复杂度约 `O(nm)`
- BF 的价值：实现简单，是理解 KMP 的参照物

### KMP 算法

#### KMP 在“省”什么？

设主串 `S`、模式串 `T`，当比较到 `S[i]` 与 `T[j]` 时失配：

- **BF**：主串起始位置后移 1 位，然后把 `T` 从头开始对齐再比（主串下标会回退）
- **KMP**：**主串指针 `i` 不回退**，只调整模式串指针 `j`（把 `T` 向右“滑动”到一个必然不会错过答案的位置）

KMP 的本质是：利用 `T` 的“自相似性”（前缀=后缀）来复用已经比对过的结果

#### 前缀、后缀、最长相等真前后缀（LPS）

- 对任意串 `X`：
  - **前缀**：`X` 的从左开始的若干字符（可取空）
  - **后缀**：`X` 的从右开始的若干字符（可取空）
  - **真前缀/真后缀**：不等于 `X` 本身的前缀/后缀
- **LPS（Longest Proper Prefix which is also Suffix）**：最长相等真前后缀长度  
  例如 `X = "ababab"`，其 LPS 长度为 `4`（"abab"）

#### 模式串 next 的定义

KMP 的关键是：失配时 **主串指针 `i` 不回溯**，只让模式串指针 `j` 依据 `next[j]` 回退。

设主串 `S = s1 s2 ... sn`，模式串 `P = p1 p2 ... pm`（**以下用 1 下标**）：

- `next[j]` 表示：当比较到 `S[i] != P[j]` 时，在不移动 `i` 的前提下，`j` 应回退到的模式串位置  
  即下一次比较变为 `S[i]` 与 `P[next[j]]`（如果 `next[j] = 0`，表示模式串第一个字符也失配）
- 直观含义：`next[j]` 给出了 `P[1..j-1]` 的“最长可复用前缀长度 + 1”的回退位置，本质就是利用“部分匹配”结果把模式串尽可能向右滑动


#### next 的求法（递推 / get_next）

```c
// 1 下标写法：P[1..m]，next[1..m]
void get_next(PString P, int next[]) {
    int i = 1, j = 0;
    next[1] = 0;
    while (i < m) {
        if (j == 0 || P[i] == P[j]) {
            ++i; ++j;
            next[i] = j;
        } else {
            j = next[j];
        }
    }
}
```

理解要点：

- `j` 始终表示：当前已知的“最长可复用前缀长度” + 1
- 当 `P[i] != P[j]` 时，不是把 `i` 回退，而是让 `j = next[j]`，继续尝试更短的可复用前缀

#### 利用 next 进行匹配

```c
// 1 下标写法：S[1..n], P[1..m]
int Index_KMP(SString S, PString P, int pos) {
    int i = pos, j = 1;
    while (i <= n && j <= m) {
        if (j == 0 || S[i] == P[j]) {
            ++i; ++j;
        } else {
            j = next[j];
        }
    }
    if (j > m) return i - m;  // 匹配成功，返回起始位置
    return 0;                // 匹配失败
}
```

#### 例子：模式串 "abaabcac" 的 next
模式串：`a b a a b c a c`（`m = 8`）

| j | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| P[j] | a | b | a | a | b | c | a | c |
| next[j] | 0 | 1 | 1 | 2 | 2 | 3 | 1 | 2 |

#### nextval（进一步减少“必然再次失配”的比较）

- 现象：若 `T[j] == T[next[j]]`，则把 `j` 跳到 `next[j]` 后，下一次比较依然会拿同样字符去比，容易产生无效比较
- `nextval` 的想法：若按 `next[j]` 回退后仍会拿“相同字符”去比较（例如 `P[j] == P[next[j]]`），则可继续回退以跳过这类必然失败的比较（教材称为 `next` 的修正值）
- 建议：
  - 先把 `next` 与 KMP 主流程写熟
  - 再在题目强调“减少比较次数/优化”时引入 `nextval`

#### 时间复杂度（为什么是 O(n+m)）

- 构造 `next/nextval`：`O(m)`
- 匹配：`O(n)`
- 直观原因：
  - `i` 单调递增不回退，总共最多走 `n` 次

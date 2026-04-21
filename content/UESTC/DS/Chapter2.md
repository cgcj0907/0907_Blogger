---
title: "第 2 章 线性表"
date: 2026-03-14
draft: false         
tags: ["UESTC", "数据结构"]
ShowToc: true            
---

<style>
  /* Chapter2 线性表结构示意图样式（内嵌 HTML 使用） */
  .ds-diagram { margin: 10px 0 18px; font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; }
  .ds-row { display: flex; align-items: center; gap: 6px; flex-wrap: wrap; }
  .ds-row.ds-col { flex-direction: column; align-items: flex-start; }
  .ds-label { font-size: 12px; color: #555; margin-bottom: 6px; }
  .ds-box { min-width: 34px; height: 30px; border: 1px solid #333; display:flex; align-items:center; justify-content:center; font-size: 12px; background: #fff; border-radius: 6px; box-shadow: 0 1px 0 rgba(0,0,0,.06); }
  .ds-box.dim { color: #999; border-style: dashed; }
  .ds-box.head { background: #f4f7ff; }
  .ds-box.ptr  { background: #fff7e6; min-width: 44px; }
  .ds-arrow { font-size: 14px; color: #333; padding: 0 2px; }
  .ds-note { font-size: 12px; color: #666; margin-top: 6px; line-height: 1.4; }
  .ds-sub { font-size: 11px; color: #666; }
  .ds-pre { margin: 6px 0 0; padding: 6px 8px; border: 1px dashed #999; color: #333; background: #fcfcfc; white-space: pre; overflow-x: auto; }
  .ds-big-arrow { font-size: 22px; font-weight: 700; letter-spacing: 2px; color: #333; }
  .ds-table { border-collapse: collapse; }
  .ds-table td { padding: 2px 6px; vertical-align: middle; }
  .ds-arrowcell { font-size: 22px; font-weight: 700; color: #333; text-align: center; min-width: 34px; }

  /* 更“节点化”的显示：data | next */
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

  /* 双向结点：prev | data | next */
  .ds-dnode {
    display: inline-flex;
    align-items: stretch;
    height: 30px;
    border: 1px solid #333;
    border-radius: 6px;
    overflow: hidden;
    background: #fff;
    box-shadow: 0 1px 0 rgba(0,0,0,.06);
  }
  .ds-dnode .prev, .ds-dnode .next {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0 10px;
    font-size: 11px;
    min-width: 54px;
    background: #fff7e6;
    color: #333;
  }
  .ds-dnode .data {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0 12px;
    font-size: 12px;
    min-width: 34px;
    border-left: 1px solid #333;
    border-right: 1px solid #333;
  }
</style>

## 线性表的定义
> 在稍微复杂的线性表中, 一个数据元素可以有若干个数据项组成, 常把数据元素称为记录, 含有大量记录的线性表又称为文件

* **表中元素的个数有限**
* 表中元素具有逻辑上的顺序性, 均为数据元素, 数据类型都相同, 具有抽象性

> 注意: 线性表是一种逻辑结构

## 线性表的表示

### 顺序表

#### 定义

> 逻辑顺序与其物理存储顺序完全一致

<div class="ds-diagram">
  <div class="ds-label">示意：顺序表（连续存储 / 数组）</div>
  <div class="ds-row ds-col" aria-label="seqlist-array-vertical">
    <div class="ds-row"><div class="ds-box head">0</div><div class="ds-box">a<sub>1</sub></div></div>
    <div class="ds-row"><div class="ds-box head">1</div><div class="ds-box">a<sub>2</sub></div></div>
    <div class="ds-row"><div class="ds-box head">2</div><div class="ds-box">a<sub>3</sub></div></div>
    <div class="ds-row"><div class="ds-box head">3</div><div class="ds-box">a<sub>4</sub></div></div>
    <div class="ds-row"><div class="ds-box head">4</div><div class="ds-box dim">∅</div></div>
    <div class="ds-row"><div class="ds-box head">5</div><div class="ds-box dim">∅</div></div>
  </div>
  <div class="ds-note"><span class="ds-sub">length</span> 记录当前元素个数；尾部预留空间用于扩展/插入</div>
</div>

1. 主要优点
* 可进行随机访问
* 存储密度高

2. 主要缺点
* 插入和删除操作效率较低, 需要移动大量元素
* 要求分配连续的存储空间

#### 基本操作的实现

1. 顺序表的初始化

* 静态分配
* 动态分配

2. 插入操作

时间复杂度
* 最好情况: 在尾标插入, O(1)
* 最坏情况: 在表头插入, O(n)
* 平均情况: O(n)

3. 删除操作

时间复杂度
* 最好情况: 删除表尾元素, O(1)
* 最坏情况: 删除表头元素, O(n)
* 平均情况: O(n)

3. 查找操作

时间复杂度
* 按值查找, O(n)
* 按索引查找, O(1)

#### 常见算法

1. 双指针
王道 P20 05.

2. 倒置
王道 P20 07. 10.

3. Boyer–Moore 投票算法
王道 P20 12.

4. 空间换时间
王道 p21 13.

### 链表
> 数据元素的存储映像称为结点, 其中存储数据元素信息的域称为数据域, 存储直接后继存储位置的域称为指针域

<div class="ds-diagram">
  <div class="ds-label">示意：单链表（data + next）</div>
  <div class="ds-row" aria-label="singly-linked-list">
    <div class="ds-box head">head</div>
    <span class="ds-arrow">→</span>
    <div class="ds-node"><span class="data">a</span><span class="next">next</span></div>
    <span class="ds-arrow">→</span>
    <div class="ds-node"><span class="data">b</span><span class="next">next</span></div>
    <span class="ds-arrow">→</span>
    <div class="ds-node"><span class="data">c</span><span class="next">NULL</span></div>
  </div>
</div>

<div class="ds-diagram">
  <div class="ds-label">示意：循环链表（尾结点 next 指回头结点）</div>
  <table class="ds-table" aria-label="circular-linked-list-2row-vertical-arrows">
    <tr>
      <td><div class="ds-box head">head</div></td>
      <td class="ds-arrowcell">→</td>
      <td><div class="ds-node"><span class="data">a</span><span class="next">next</span></div></td>
    </tr>
    <tr>
      <td class="ds-arrowcell">↑</td>
      <td></td>
      <td class="ds-arrowcell">↓</td>
    </tr>
    <tr>
      <td><div class="ds-node"><span class="data">c</span><span class="next">next</span></div></td>
      <td class="ds-arrowcell">←</td>
      <td><div class="ds-node"><span class="data">b</span><span class="next">next</span></div></td>
    </tr>
  </table>
  <div class="ds-note">用“尾指回头”体现循环结构：遍历不会遇到 NULL，而是回到起点继续</div>
</div>

<div class="ds-diagram">
  <div class="ds-label">示意：双向链表（prev + data + next）</div>
  <div class="ds-row" aria-label="doubly-linked-list">
    <div class="ds-box dim">NULL</div>
    <span class="ds-arrow">←</span>
    <div class="ds-dnode"><span class="prev">prev</span><span class="data">a</span><span class="next">next</span></div>
    <span class="ds-arrow">↔</span>
    <div class="ds-dnode"><span class="prev">prev</span><span class="data">b</span><span class="next">next</span></div>
    <span class="ds-arrow">↔</span>
    <div class="ds-dnode"><span class="prev">prev</span><span class="data">c</span><span class="next">next</span></div>
    <span class="ds-arrow">→</span>
    <div class="ds-box dim">NULL</div>
  </div>
</div>

<div class="ds-diagram">
  <div class="ds-label">示意：静态链表（用数组模拟：data + cur/next 下标）</div>
  <div class="ds-row"><div class="ds-box head">idx</div><div class="ds-box head">data</div><div class="ds-box head">cur</div></div>
  <div class="ds-row ds-col" aria-label="static-linked-list-rows-vertical">
    <div class="ds-row"><div class="ds-box">0</div><div class="ds-box dim">—</div><div class="ds-box">5</div></div>
    <div class="ds-row"><div class="ds-box">5</div><div class="ds-box">a</div><div class="ds-box">8</div></div>
    <div class="ds-row"><div class="ds-box">8</div><div class="ds-box">b</div><div class="ds-box">2</div></div>
    <div class="ds-row"><div class="ds-box">2</div><div class="ds-box">c</div><div class="ds-box">0</div></div>
  </div>
  <div class="ds-note">
    <span class="ds-sub">cur</span> 存“下一个结点所在数组下标”；常用 <span class="ds-sub">0</span> 表示空指针<span class="ds-sub">0 号</span>位置也常用于保存备用链（空闲结点表）信息
  </div>
</div>

1. 单链表
各操作时间复杂度
* 求表长操作: O(n)
* 按序号查找节点: O(n)
* 按值查找表节点: O(n)
* 插入节点操作(含查找): O(n)
* 删除节点操作(含查找): O(n)
* 头插法建立单链表: O(n)
* 尾插法建立单链表: O(n)

2. 循环链表

3. 双向链表

4. 静态链表
* 含备用链
* 不含备用链

#### 常见算法

1. 快慢指针
王道 p44 15. 16. 17. 20.

2. 公共节点
王道 p43 05. p45 18.

3. 空间换时间
王道 p45 19.

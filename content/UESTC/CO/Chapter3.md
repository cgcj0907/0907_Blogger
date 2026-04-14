---
title: "第 3 章 运算方法和运算部件"
date: 2026-03-30
draft: false         
tags: ["UESTC", "计算机组成原理"]
ShowToc: true            
---

## C 语言中涉及的运算
1. 按位运算：或运算（｜）、与运算（&）、取反运算（～）、异或运算（^）
2. 逻辑运算：或运算（||）、与运算（&&）、非运算（！）
3. 移位运算：
	* 逻辑移位
	* 算术移位：右移高位补符号
4. 位扩展和位截断运算
	* 0 扩展
	* 符号扩展
## 带标志加法器
* 溢出标志 OF(Overflow Flag) : <code>OF = C<sub>n</sub> ⊕ C<sub>n-1</sub></code>
* 符号标志 SF(Sign Flag) : <code>SF = F<sub>n-1</sub></code>
* 零号标志 ZF(Zero Flag) ：ZF = 1，当且仅当 F = 0
* 进位/借位标志 CF(Carry Flag) ：<code>CF = C<sub>out</sub> ⊕ C<sub>in</sub></code>
## 定点数运算
### 无符号数和补码加减法运算
![Compound addition](/UESTC/CO/Chapter3_Compound_1.webp)
><span style="text-decoration:overline;">Y</span> 即 Y 各位取反

><code>CF = C<sub>in</sub> ⊕ C<sub>out</sub></code>，CF 对于补码加减法运算无实际意义
* 无符号数(X 和 Y 就是 x 和 y 的二进制表示)
	* 加法 : <code>X + Y, C<sub>in</sub> = 0</code>
	* 减法 : <code>X + <span style="text-decoration:overline;">Y</span> + 1, C<sub>in</sub> = 1</code>
* 补码(X 和 Y 就是 x 和 y 的补码表示)
	* 加法 : <code>X + Y, C<sub>in</sub> = 0</code>
	* 减法 : <code>X + <span style="text-decoration:overline;">Y</span> + 1, C<sub>in</sub> = 1</code>
### 原码乘法运算
#### 原码一位乘法
![Unsigned multiplication](/UESTC/CO/Chapter3_Usigned_1.webp)
![Unsigned multiplication](/UESTC/CO/Chapter3_Usigned_2.webp)
### 补码乘法运算
![Two's complement multiplication](/UESTC/CO/Chapter3_TwoC_1.webp)
![Two's complement multiplication](/UESTC/CO/Chapter3_TwoC_2.webp)

#### 布斯（Booth）乘法推导（补码乘法的核心思想）

布斯乘法的目标：把乘数中连续的 <code>1</code>（一段“1 串”）改写成“加一次 + 减一次”的形式，从而减少加法次数。

- “1 串”恒等式（把连续求和化为差）：
  <p><code>&Sigma;<sub>i=k</sub><sup>m</sup> 2<sup>i</sup> = 2<sup>m+1</sup> - 2<sup>k</sup></code></p>
  因此若乘数 <code>Y</code> 的二进制中在 <code>[k, m]</code> 出现一段连续的 <code>1</code>，则对应部分乘积为：
  <p><code>X &times; (&Sigma;<sub>i=k</sub><sup>m</sup> 2<sup>i</sup>) = (X &lt;&lt; (m+1)) - (X &lt;&lt; k)</code></p>
  即“在 1 串的起点减一次，在 1 串的终点后加一次”（或等价的加/减顺序约定）。

- 用“相邻位变化”统一表达（Booth 重编码）：
  - 设乘数为 <code>Y = (y<sub>n-1</sub> ... y<sub>1</sub> y<sub>0</sub>)</code>，并在最低位右侧附加一位 <code>y<sub>-1</sub> = 0</code>（等价于引入 <code>Q<sub>-1</sub></code>）。
  - 定义重编码系数：
    <p><code>d<sub>i</sub> = y<sub>i-1</sub> - y<sub>i</sub>, &nbsp; d<sub>i</sub> &in; { -1, 0, 1 }</code></p>
  - 对于无符号表示（或对补码先做符号扩展后同理处理），可以把乘数写成：
    <p><code>Y = &Sigma;<sub>i=0</sub><sup>n</sup> d<sub>i</sub> &middot; 2<sup>i</sup></code></p>
    直观解释：<code>y<sub>i</sub></code> 与 <code>y<sub>i-1</sub></code> 相同（00/11）表示“1 串内部或 0 区间”，系数为 0；发生 <code>01</code> / <code>10</code> 跳变时就对应“1 串边界”，系数为 ±1。
  - 因而乘积可写为：
    <p><code>X &times; Y = &Sigma;<sub>i=0</sub><sup>n</sup> d<sub>i</sub> &middot; (X &lt;&lt; i)</code></p>

- 与常用 Booth 判别规则的对应关系（看两位：<code>(y<sub>i</sub>, y<sub>i-1</sub>)</code>）：
  <table>
    <thead>
      <tr><th>(y<sub>i</sub>, y<sub>i-1</sub>)</th><th>d<sub>i</sub> = y<sub>i-1</sub> - y<sub>i</sub></th><th>对部分积的动作</th></tr>
    </thead>
    <tbody>
      <tr><td>00</td><td>0</td><td>不加不减</td></tr>
      <tr><td>01</td><td>+1</td><td>加 <code>+ (X &lt;&lt; i)</code></td></tr>
      <tr><td>10</td><td>-1</td><td>减 <code>- (X &lt;&lt; i)</code></td></tr>
      <tr><td>11</td><td>0</td><td>不加不减</td></tr>
    </tbody>
  </table>

- 为什么适用于补码乘法（要点）：
  - 补码是带符号表示，实际实现中对乘数会做**符号扩展**，并对（部分积/乘数/附加位）做**算术右移**；
  - Booth 的“看相邻两位决定加/减”的规则本质上只依赖于 <code>0/1</code> 跳变位置，因此对补码（含负数）同样成立。
### 原码除法运算
>符号位单独运算
>定点整数，被除数扩展高位添加 0；定点小数，被除数扩展低位添加 0
><code>Q<sub>n</sub> = 1</code> 时，定点整数除法和定点小数除法均溢出，浮点数除外（可右规）
![Division](/UESTC/CO/Chapter3_Div.webp)
#### 恢复余数除法
![Division](/UESTC/CO/Chapter3_Div_Reset_1.webp)
![Division](/UESTC/CO/Chapter3_Div_Reset_2.webp)
#### 不恢复余数除法
![Division NoReset](/UESTC/CO/Chapter3_Div_Noreset_1.webp)
![Division NoReset](/UESTC/CO/Chapter3_Div_Noreset_2.webp)
### 补码除法运算
>被除数需要进行符号扩展 
![Division Two's complement rules](/UESTC/CO/Chapter3_TwoC_Div_Rules.webp)
#### 恢复余数除法
>若被除数与除数同号，则 Q 中是真正的商；否则，将 Q 中的数值求补后作为真正的商
![Division Two's complement division with reset](/UESTC/CO/Chapter3_TwoC_Div_Reset.webp)
#### 不恢复余数除法
>商的修正：最后一次 Q 寄存器左移一位，将最高位 <code>Q<sub>n</sub></code> 移出，并在最低位置上商 <code>Q<sub>0</sub></code>
>余数的修正：若余数符号同被除数符号，则不需修正，余数在 R 中；否则，按下列规定进行修正：当被除数和除数符号相同时，最后余数加除数；否则，最后余数减除数


![Division Two's complement division with  no reset](/UESTC/CO/Chapter3_TwoC_Div_Noreset.webp)
## 浮点数运算的精度和舍入
>保护位 G（警戒位）：紧跟在有效位后面的第 1 位
>舍入位 R ：紧跟在有效位后面的第 2 位
>粘位 S ：后面所有位的“或”结果，即舍入右边只要有 1，粘位则置为 1
1. 就近舍入到偶数
```
舍入规则：  
G=0 → 不进位  
G=1 且（R=1 或 S=1）→ 进位  
G=1 且 R=0 且 S=0 → 向偶数舍入
```
2. 朝 <code>+&infin;</code> 方向舍入
3. 朝 <code>-&infin;</code> 方向舍入
4. 朝 0 方向舍入
![Result rounding](/UESTC/CO/Chapter3_Result_Rounding.webp)

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

---
title: "第 2 章 数据的机器级表示"
date: 2026-03-30
draft: false         
tags: ["UESTC", "计算机组成原理"]
ShowToc: true            
---
# IEEE754
![IEEE754](/UESTC/CO/Chapter2_IEEE754_1.webp)
![IEEE754](/UESTC/CO/Chapter2_IEEE754_2.webp)
# ASCII
>American Standard Code for International InterInterchange

![ASCII](/UESTC/CO/Chapter2_ASCII_1.webp)
![ASCII](/UESTC/CO/Chapter2_ASCII_2.webp)
# 数据的宽度和存储
## 主存容量使用的单位
<table>
  <tbody>
    <tr><td>1 KB (kilobyte)</td><td>=</td><td>2<sup>10</sup> 字节</td></tr>
    <tr><td>1 MB (megabyte)</td><td>=</td><td>2<sup>20</sup> 字节</td></tr>
    <tr><td>1 GB (gigabyte)</td><td>=</td><td>2<sup>30</sup> 字节</td></tr>
    <tr><td>1 TB (terabyte)</td><td>=</td><td>2<sup>40</sup> 字节</td></tr>
    <tr><td>1 PB (petabyte)</td><td>=</td><td>2<sup>50</sup> 字节</td></tr>
    <tr><td>1 EB (exabyte)</td><td>=</td><td>2<sup>60</sup> 字节</td></tr>
    <tr><td>1 ZB (zettabyte)</td><td>=</td><td>2<sup>70</sup> 字节</td></tr>
    <tr><td>1 YB (yottabyte)</td><td>=</td><td>2<sup>80</sup> 字节</td></tr>
  </tbody>
</table>
## 主频和带宽使用的单位
<table>
  <tbody>
    <tr><td>比特/秒</td><td>:</td><td>b/s 或 bps</td></tr>
    <tr><td>1 kb/s</td><td>=</td><td>10<sup>3</sup> b/s = 1000 bps</td></tr>
    <tr><td>1 Mb/s</td><td>=</td><td>10<sup>6</sup> b/s = 1000 kb/s</td></tr>
    <tr><td>1 Gb/s</td><td>=</td><td>10<sup>9</sup> b/s = 1000 Mb/s</td></tr>
    <tr><td>1 Tb/s</td><td>=</td><td>10<sup>12</sup> b/s = 1000 Gb/s</td></tr>
  </tbody>
</table>

## 硬盘和文件使用的单位
![Unit](/UESTC/CO/Chapter2_Unit_1.webp)
![Unit](/UESTC/CO/Chapter2_Unit_2.webp)
# 数据的存储和排列顺序
* 大端方式 : 将数据的最高有效字节 MSB 存放在低地址单位中
* 小端方式 : 将数据的最低有效字节 LSB 存放在低地址单位中
>示例 : 机器数 01234567 H
![Storage](/UESTC/CO/Chapter2_Storage.webp)

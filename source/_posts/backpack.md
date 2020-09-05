---
title: 背包进阶知识讲解
date: 2018-03-02 15:13:49
mathjax: true
tags:
- 算法讲解
- 动态规划
- 背包
categories:
- 动态规划
- 背包
---

# 背包进阶知识讲解

目录

- 二进制拆分
- 路径记录

<!-- more -->

## 二进制拆分

在处理多重背包或完全背包时，朴素的做法会循环判断k，求出当前物品最优的选择个数，这使程序跑地很慢。而二进制拆分就解决了这一问题。

我们知道，将任意十进制数转化为二进制数后，可用0和1表示。

从又到左的每一个数位，分别表示$2^0$，$2^1$，$2^2$，$2^3$......

所以，只需使用$1$，$2$，$4$，$8$......就可以表示所有十进制数。因此，我们将物品个数$c$转化为$1$，$2$，$4$......$2^k$，$c-2^{(k+1)}+1$。从而优化时间复杂度。

## 路径记录

如果题目要求输出选择的物品编号，就需要记录路径。


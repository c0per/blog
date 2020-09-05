---
title: 莫比乌斯反演入门
date: 2019-04-11 18:52:49
tags:
- 算法讲解
- 莫比乌斯反演
categories:
- 数学
- 莫比乌斯反演
---

# 莫比乌斯反演入门

莫比乌斯反演描述了 $2$ 个数论函数之间的关系和互相转化。

在推式子的时候非常有用。

<!-- more -->

## 狄利克雷卷积

对于 $2​$ 个数论函数 $f(n), g(n)​$ ，他们的狄利克雷卷积定义为：

$$h(n) = \sum_{d|n} f(d)*g(\frac {n} {d})$$

就是说 $2$ 个数论函数卷积计算后变成了 $1$ 个新的数论函数啦。

## 莫比乌斯函数

根据唯一分解定理，正整数 $x = ​\prod_{i=1}^{k} p_{i}^{c_i} (\forall p_i \in prime)$

$$\mu(x) = \begin{cases}
1 & x=1\\
(-1)^k & \forall c_i \le 1\\
0 & \exists c_i \gt 1\\
\end{cases}$$

### 性质1

$$\sum_{d|x} \mu(d) = [x=1]$$

比如：

$$\sum_{i=1}^n \sum_{j=1}^m[\gcd(i,j)=1] \qquad (n \le m)$$

$$=\sum_{i=1}^n \sum_{j=1}^m \sum_{d|\gcd(i,j)} \mu(d)$$

$$=\sum_{d=1}^n \lfloor \frac{n}{d} \rfloor \lfloor \frac{m}{d} \rfloor \mu(d)$$

### 性质2

$$\sum_{d|x} \frac{\mu(d)}{d} = \frac{\phi(x)}{x}$$

### 例题

- [[POI2007]ZAP-Queries](https://www.luogu.org/problemnew/show/P3455)

- [YY的GCD](https://www.luogu.org/problemnew/show/P2257)

- [[国家集训队]Crash的数字表格](https://www.luogu.org/problemnew/show/P1829)

- [[HAOI2011]Problem b](https://www.luogu.org/problemnew/show/P2522)

## 莫比乌斯反演

对于 $2$ 个数论函数 $f(x), g(x)$ 。

如果满足：

$$f(x) = \sum_{d|x} g(d)$$

那么：

$$g(x) = \sum_{d|x} \mu(d) f(\frac{x}{d})$$

反演还有另一种形式：

$$f(x) = \sum_{x|d} g(d)$$

那么：

$$g(x) = \sum_{x|d} \mu(\frac{d}{x}) f(d)$$

### 例题

- [[SDOI2015]约数个数和](https://www.luogu.org/problemnew/show/P3327)
---
title: ST表
date: 2018-02-16 14:37:54
tags:
- ST表
- 算法讲解
categories:
- 基础算法
- ST表
---

# ST表

## 算法简介

$ST$表是一张表，采用了倍增的原理，记录从$i$到$i+2^j-1$的信息。

通常，$ST$表被用来解决$RMQ$问题。

<!-- more -->

$ST$表只需$O(nlog_n)$的预处理，便可实现$O(1)$的查询。

## 核心代码

```c++
void init()
{
    for(int i=1;i<=n;++i)
        st[i][0]=a[i];
    for(int j=1;j<48;++j)
        for(int i=1;i+(1<<j-1)<=n;++i)
            f[i][j]=min(f[i][j-1],f[i+(1<<j-1)][j-1]);
    for(int i=2;i<=n;++i)
        lg2[i]=lg2[i>>1]+1;
}
void rmq(int l,int r)
{
    int t=lg2[r-l-1];
    return min(f[l][t],f[r-(1<<t)+1][t]);
}
```

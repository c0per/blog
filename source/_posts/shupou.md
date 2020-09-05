---
title: 树链剖分
date: 2018-02-16 14:24:09
tags:
- 算法讲解
- 树链剖分
categories:
- 数据结构
- 树链剖分
---

# 数链剖分

## 简介

平时，我们总会遇到许多树上的问题，比如求$LCA$，树上路径等等。

通常，他们需要$O(log_N)$甚至更长的时间，但我们可以发现，只要他们在同一条链上，所需的时间就会大大减少。

这就是树链剖分的思路，将一棵树**剖成尽量少的链**，从而优化时间。

<!-- more -->

## 算法细节

那么，如何将一棵我们可以**把树链分成重链和轻链**，并记录每个节点被深搜到的时间戳，使用线段树进行操作。

---

记$size[i]$表示以$i​$为根的子树的节点数

$dep[i]​$为节点i的深度

$top[i]$表示节点i所在链上$dep​$值最小的点的编号（重链顶点）

$maxson[i]​$表示节点i的重儿子

$dfn[i]$表示深搜到点i的时间戳（便于使用线段树维护）

$fa[i]$表示节点$i$的父节点编号

**重儿子：$size$最大的那个子节点**

**轻儿子：其他子节点**

**重边：根节点到重儿子的边**

**轻边：根节点到轻儿子的边**

**重链：重边组成的链**

**轻链：轻边组成的链**

每个节点都有一个重链和若干个轻链，这样我们得到的树链数就是最少的了。

## 核心代码

```c++
void hld()
{
    p[1].dep=1;
    p[1].fa=1;
    dfs1(1);//预处理
    dfs2(1,1);//树链剖分
    build(1,1,n);//线段树维护
}
void dfs1(int pos)
{
    int maxsz=0;
    p[pos].size=1;
    p[pos].maxson=0;//重儿子的编号
    for(int i=head[pos];i;i=e[i].next)
    {
        if(e[i].t==p[pos].fa)
            continue;
        p[e[i].t].dep=p[pos].dep+1;
        p[e[i].t].fa=pos;//记录父节点
        dfs1(e[i].t);
        if(p[e[i].t].size>maxsz)
        {
            maxsz=p[e[i].t].size;
            p[pos].maxson=e[i].t;
        }
        p[pos].size+=p[e[i].t].size;
    }
}
void dfs2(int pos,int tp)
{
    p[pos].dfn=++idx;
    p[pos].top=tp;
    ref[idx]=pos;//ref[i]=dfn为i的节点的编号
    if(p[pos].maxson)
        dfs2(p[pos].maxson,tp);//处理重链
    else return;
    for(int i=head[pos];i;i=e[i].next)//处理其他轻链
        if(e[i].t!=p[pos].maxson&&e[i].t!=p[pos].fa)
            dfs2(e[i].t,e[i].t);
}
```

### 练习题

[Luogu P2590 [ZJOI2008]树的统计](https://www.luogu.com.cn/problem/P2590)
[Luogu P3178 [HAOI2015]树上操作](https://www.luogu.com.cn/problem/P3178)

## 求LCA

将一颗树剖分成一条条链后，很明显，可以快速求出$lca$。

- 如果当前查询的$2$个节点所在重链的$top$值相同（在同一条重链上），答案就是深度较浅的节点。
- 否则，比较所在重链顶点（$top$）的深度，将结果较深的节点更改为重链顶点的父节点。

可以轻松写出递归代码。

```c++
int getlca(int a,int b)
{
    if(t[a].top==t[b].top)
        return t[a].dep<t[b].dep?a:b;
    if(t[t[a].top].dep<t[t[b].top].dep)
        return getlca(a,t[t[b].top].fa);
    else
        return getlca(t[t[a].top].fa,b);
}
```

### 练习题

[P3258 [JLOI2014]松鼠的新家](https://www.luogu.com.cn/problem/P3258)

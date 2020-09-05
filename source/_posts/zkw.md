---
title: EK费用流，zkw费用流入门
date: 2019-01-07 12:51:23
tags:
- 算法讲解
- 费用流
categories:
- 网络流
- 费用流
---

# EK费用流，zkw费用流入门

## 什么是费用流？

在普通的网络流中，每一条边拥有相应的费用，即没有一单位的流量，就会产生一单位的费用。

在满足**最大流的前提下**，满足总费用最小，即$\sum_{i \in E}f_i$最大，$\sum_{i \in E}f_i*c_i$最小。

<!-- more -->

## EK算法

使用EK算法就可以解决啦，增广时在惨量网络上求出以费用为边权的，从$S$到$T$的最短路，并记录最短路上可行流的最小值，累加答案。

## zkw费用流

### 算法原理

每次都跑一边最短路实在是太慢啦qwq！我们可不可以用类似KM算法的思想来解决呢？

$zkw$巨巨：可以

这种神奇算法就是$zkw$费用流。

观察跑完最短路算法后的$dis​$数组。

如果存在一条从$u​$到$v​$，边权为$c​$的边。那么，$dis_v-dis_u \le c​$。

化一下这个柿子：

$$dis_v-dis_u-c \le 0$$

$$dis_u-dis_v+c \ge 0$$

当且仅当，该边在$u$到$v$最短路上时，取等号。

为了满足最小费用，我们只在最短路上增广就好啦！

```c++
int dfs(int x,int flow)
{
    v[x]=1;
    if(x==T)
    {
        maxflow+=flow;
        cost;//想办法修改cost
        return flow;
    }
    int rest=flow,k;
    for(int i=head[x];i&&rest;i=e[i].nxt)
        if(!v[e[i].t]&&e[i].w&&d[x]-d[e[i].t]+e[i].c==0)//保证在最短路上
        {
            k=dfs(e[i].t,std::min(rest,e[i].w));
            e[i].w-=k;
            e[i^1].w+=k;
            rest-=k;
        }
    return flow-rest;
}
```

{% asset_image stop.jpg %}

---

说得好，最大流的性质呢？？？

为了满足流量最大的前提，我们还要**在没有那么优秀的边上增广**。

**所以，zkw算法的大体思路是，优先在最短路上增广，其次在次短路上增广，再其次在次次短路上增广。**

**通过将一条条边纳入当前可增广图，我们不需要一遍遍跑最短路啦。**

{% asset_image order.jpg %}

按照边的优秀程度，我们一边一条条将边加入图中，一边增广流量。

---

说得好，怎么做呢？

上文中满足的$dis_u-dis_v+c = 0$的边应为图中最短路的边。如果只走这些边，只有部分点可达。

为了扩大我们可增广的范围，我们需要将一些**起点可达，终点不可达**的边加入图中。

{% asset_image edge.jpg %}

设$dis_u-dis_v+c​$为$\Delta​$，我们只需将$dis_u​$减去$\Delta​$，即可满足$dis_u-dis_v+c = 0​$。

为了满足之前可达的点仍然可达，将所有可达点的$dis$都减去$\Delta$，可行边仍然可行。

同时，不能一次加入的边过多使得答案不优秀。所以令$\Delta=\min(dis_u-dis_v+c)​$。

```c++
bool mod()
{
    long long delta=inf;
    for(int x=S;x<=T;++x)
        if(v[x])
            for(int i=head[x];i;i=e[i].nxt)
                if(e[i].w&&!v[e[i].t])
                    delta=std::min(delta,d[x]-d[e[i].t]+e[i].c);
    if(delta==inf)
        return 0;
    for(int x=S;x<=T;++x)
        if(v[x])
            d[x]-=delta;
    return 1;
}

```

---

每次dfs到$T$时，费用累加$-dis_S*flow$，其中$-dis_S$已经累加过的$\Delta$。

### 代码实现

```c++
ll dfs(int x,ll flow)
{
    v[x]=1;
    if(x==T)
    {
        cost+=-d[S]*flow;
        maxflow+=flow;
        return flow;
    }
    ll rest=flow,k;
    for(int i=head[x];i&&rest;i=e[i].nxt)
        if(!v[e[i].t]&&e[i].w&&d[x]-d[e[i].t]+e[i].c==0)
        {
            k=dfs(e[i].t,std::min(rest,(ll)e[i].w));
            e[i].w-=k;
            e[i^1].w+=k;
            rest-=k;
        }
    return flow-rest;
}

bool mod()
{
    ll delta=inf;
    for(int x=S;x<=T;++x)
        if(v[x])
            for(int i=head[x];i;i=e[i].nxt)
                if(e[i].w&&!v[e[i].t])
                    delta=std::min(delta,d[x]-d[e[i].t]+e[i].c);
    if(delta==inf)
        return 0;
    for(int x=S;x<=T;++x)
        if(v[x])
            d[x]-=delta;
    return 1;
}

void zkw()
{
    do
        do
            memset(v,0,sizeof(v));
        while(dfs(S,inf));
    while(mod());
    printf("%lld %lld",maxflow,cost);
}
```

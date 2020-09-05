---
title: tarjan算法与连通分量
date: 2018-08-03 14:44:47
tags:
- 算法讲解
- 图论
categories:
- 图论
---

# tarjan算法与连通分量

这篇博客记录了各种tarjan算法和各种连通分量的模板。**未完待续**

<!-- more -->

## 强连通分量

### 求法

强连通分量指**有向图**中任意两点可相互到达的极大子图，可以很简单的用$tarjan$求出，不多叙述。

$dfs$一次将节点入栈，对未访问的子节点依次$dfs$并维护可由子节点到达的最小$dfn$，用已访问的子节点更新自己的$low$。

若$dfn[x] = low[x]$则代表x节点为当前强连通分量“最上面”的节点。因为当前节点无法访问$dfn​$比自己小的节点，即不与“上面”的其他节点连通。将栈弹出至当前节点。

```c++
void tarjan(int x)
{
    dfn[x]=low[x]=++tim;
    stack[++top]=x;ins[x]=true;
    for(int i=head[x];i;i=e[i].nxt)
    {
        if(!dfn[e[i].t])
        {
            tarjan(e[i].t);
            low[x]=std::min(low[x],low[e[i].t]);
        }
        else if(ins[e[i].t])
            low[x]=std::min(low[x],dfn[e[i].t]);
    }
    if(low[x]==dfn[x])
    {
        ++num;
        do
        {
            c[stack[top]]=num;
            ins[stack[top--]]=false;
        }while(stack[top+1]!=x)
    }
}

int main()
{
for(int i=1;i<=n;++i)
    if(!dfn[i])
        tarjan(i);
}
```

将全图的$tarjan$跑完后，$c[x]$表示节点$x$所属的强连通分量的编号，$num$为强连通分量的个数。

### 缩点

将一个强连通分量看作一个点，所构成的图必然是一条链，一棵树，或一个环（边的方向不相同的环）。

我们可以在这张图上操作。

## 桥

**无向连通图**中，如果删除某边后，图变成不连通，则称该边为桥。

一样进行$dfs$。对于每个未访问子节点$y$，若存在$low[y] \gt dfn[x]​$，则当前边为桥。因为子节点无法不通过当前节点来访问到更“以前”的节点，所以当前边是该子节点“向上”的必经之路，删除后图不连通。（图一）

而当$low[y] \le dfn[x]$时，当前边显然不为桥。（图二）

{% asset_image bridge.png 桥的判定 %}

### 求法

```c++
void tarjan(int x,int fa)
{
    dfn[x]=low[x]=++tim;
    for(int i=head[x];i;i=e[i].nxt)
    {
        if(e[i].t==fa)continue;//排除连向父亲的回边，不然就求不出桥啦。
        if(!dfn[e[i].t])
        {
            tarjan(e[i].t,x);
            low[x]=std::min(low[x],low[e[i].t]);
            if(low[e[i].t]>dfn[x])
                bridge[i]=bridge[i^1]=true;
                //临接表若从2号开始建边，则i和i^1为双向边。
        }
        else low[x]=std::min(low[x],dfn[e[i].t]);
    }
}
```

## 边双连通分量

如果我们将桥删去，则图被分为许多连通块。每一个连通块内的任意两点可通过**两**条没有重复边的路径互相到达（可以有重复的点）。这样的子图被称为边**双**连通分量。

{% asset_image v-DCC.png 边双连通分量 %}

### 求法

方法很直观，跑一遍$tarjan$将所有的桥删去，再$dfs$统计剩余的连通块即可。

```c++
void tarjan(int x,int fa)
{
    dfn[x]=low[x]=++tim;
    for(int i=head[x];i;i=e[i].nxt)
    {
        if(e[i].t==fa)continue;//排除连向父亲的回边，不然就求不出桥啦。
        if(!dfn[e[i].t])
        {
            tarjan(e[i].t,x);
            low[x]=std::min(low[x],low[e[i].t]);
            if(low[e[i].t]>dfn[x])
                e[i].ban=e[i^1].ban=true;
                //临接表若从2号开始建边，则i和i^1互为双向边。
        }
        else low[x]=std::min(low[x],dfn[e[i].t]);
    }
}

void dfs(int x)
{
    vis[x]=true;
    c[x]=num;
    for(int i=head[x];i;i=e[i].nxt)
    	if(!e[i].ban&&!vis[e[i].t])
            dfs(e[i].t);
}

int main()
{
    tarjan(1,0);
    for(int i=1;i<=n;++i)
        if(!vis[i])
        {
            ++num;
            dfs(i);
        }
}
```

### 缩点

如果我们将一个边双连通块看作一个点，桥看作一个边，重新建图后可以得到一棵树。

同样将节点押入栈中，若出现子节点$y$使$low[y]>dfn[x]$。则出现了一个边双连通块，出栈直到$y$节点被弹出。

{% asset_image v-DCC2.png 边双缩点 %}

```c++
void tarjan(int x,int fa)
{
    dfn[x]=low[x]=++tim;
    for(int i=head[x];i;i=e[i].nxt)
    {
        if(e[i].t==fa)continue;
        if(!dfn[e[i].t])
        {
            tarjan(e[i].t,x);
            low[x]=std::min(low[x],low[e[i].t]);
            if(low[e[i].t]>dfn[x])
                e[i^1].ban=e[i].ban=1;
        }
        else low[x]=std::min(low[x],dfn[e[i].t]);
    }
}

void dfs(int x)
{
    vis[x]=true;
    c[x]=num;
    for(int i=head[x];i;i=e[i].nxt)
    {
        if(e[i].ban)
        {
            if(vis[e[i].t])
            {
                add(c[x],c[e[i].t]);
                add(c[e[i].t],c[x]);
            }
            continue;
        }
        if(vis[e[i].t])continue;
        dfs(e[i].t);
    }
}

int main()
{
    tarjan(1,0);
    num=n;
    for(int i=1;i<=n;++i)
            if(!vis[i])
            {
                ++num;
                dfs1(i);
            }
}
```

$num$为边双的数量，从$n+1$开始编号。

## 割点

将一张联通图中的任意一割点删除后，图将不再连通。这样的点就是割点。

### 求法

```c++
void tarjan(int x)
{
    int f=0;
    dfn[x]=low[x]=++ti;
    for(int i=head[x];i;i=e[i].nxt)
    {
        if(!dfn[e[i].t])
        {
            tarjan(e[i].t);
            low[x]=std::min(low[x],low[e[i].t]);
            if(low[e[i].t]>=dfn[x])
            {
                ++f;
                if(f>1||x!=rt)
                {
                    if(!cut[x])++ccnt;
                    cut[x]=true;
                }
            }
        }
        else low[x]=std::min(low[x],dfn[e[i].t]);
    }
}
```

## 点双连通分量

将图中的每一个割点删掉后，原图变成许多子图。每一个子图就是一个点双连通分量。

可以发现，每一个连通分量中的任意两点，都有2条没有相同点的路径相连。

需要注意的是，每个割点可能在多个点双连通分量中出现，要根据题意进行判断。

### 求法

### 缩点

和边双连通分量一样，我们将一个点双连通分量看成一个点，重新建图。
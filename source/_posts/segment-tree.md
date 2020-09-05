---
title: 线段树小结
date: 2020-07-29 17:00:00
tags:
- 线段树
categories:
- 数据结构
- 线段树
---

# 线段树小结

整理归纳一下 OI 竞赛中的线段树考法，持续更新。

<!-- more -->

## 普通的线段树

一般来说，他只支持单点修改、区间查询。

```c++
struct set_t {
    int c[maxn << 2];
    void push_up(int k) {
        c[k] = c[k << 1] + c[k << 1 | 1];
    }
    void init(int k, int l, int r) {
        if (l == r) {
            c[l] = a[l];
            return;
        }
        int mid = (l + r) >> 1;
        init(k << 1, l, mid);
        init(k << 1 | 1, mid + 1, r);
        push_up(k);
    }
    int query(int k, int l, int r, int L, int R) {
        if (L <= l && r <= R)
            return c[k];
        int mid = (l + r) >> 1;
        int re = 0;
        if (L <= mid) re += query(k << 1, l, mid, L, R);
        if (R > mid) re += query(k << 1 | 1, mid + 1, r, L, R);
        return re;
    }
    void mod(int k, int l, int r, int pos, int x) {
        if (l == r) {
            c[k] += x;
            return;
        }
        int mid = (l + r) >> 1;
        if (pos <= mid) mod(k << 1, l, mid, pos, x);
        else mod(k << 1 | 1, mid + 1, r, pos, x);
        push_up(k);
    }
}t;
```

之所以不支持区间修改，是因为每次修改必须要访问到叶子节点，所以会有$O(nlog_{2}n)$的复杂度。

### 练习题

[Luogu P4588 [TJOI2018]数学计算](https://www.luogu.com.cn/problem/P4588)

## Lazy Tag 线段树

这个才是我们通常所说的线段树。

他支持区间修改、区间查询。

```c++
struct set_t {
    int c[maxn << 2], tag[maxn << 2];
    void push_up(int k) {
        c[k] = c[k << 1] + c[k << 1 | 1];
    }
    void push_down(int k, int l, int r) {
        int mid = (l + r) >> 1;
        tag[k << 1] += tag[k];
        tag[k << 1 | 1] += tag[k];
        c[k << 1] += tag[k] * (mid - l + 1);
        c[k << 1 | 1] += tag[k] * (r - mid);
        tag[k] = 0;
    }
    void init(int k, int l, int r) {
        if (l == r) {
            c[l] = a[l];
            return;
        }
        int mid = (l + r) >> 1;
        init(k << 1, l, mid);
        init(k << 1 | 1, mid + 1, r);
        push_up(k);
    }
    int query(int k, int l, int r, int L, int R) {
        if (L <= l && r <= R)
            return c[k];
        push_down(k, l, r);
        int mid = (l + r) >> 1;
        int re = 0;
        if (L <= mid) re += query(k << 1, l, mid, L, R);
        if (R > mid) re += query(k << 1 | 1, mid + 1, r, L, R);
        return re;
    }
    void mod(int k, int l, int r, int pos, int x) {
        if (l == r) {
            c[k] += x;
            return;
        }
        push_down(k, l, r);
        int mid = (l + r) >> 1;
        if (pos <= mid) mod(k << 1, l, mid, pos, x);
        else mod(k << 1 | 1, mid + 1, r, pos, x);
        push_up(k);
    }
    void add(int k, int l, int r, int L, int R, int x) {
        if (L <= l && r <= R) {
            c[k] += (r - l + 1) * x;
            tag[k] += x;
            return;
        }
        push_down(k, l, r);
        int mid = (l + r) >> 1;
        if (L <= mid) add(k << 1, l, mid, L, R, x);
        if (R > mid) add(k << 1 | 1, mid + 1, r, L, R, x);
        push_up(k);
    }
}t;
```

之所以称为 Lazy 的 Tag，是因为只有当需要访问儿子节点时，才会将本节点的标记下放（`push_down()`）。

对于一个区间操作，只有区间两边的叶子节点才有被访问的可能，其他中间的节点 Tag 标记后就不再继续往下。

这样，保证每次修改、查询的复杂度为$log_{2}n$。

代码中需要注意及时`push_down()`和`push_up()`。

### 练习题

[Luogu P3372 【模板】线段树 1](https://www.luogu.com.cn/problem/P3372)

[Luogu P4198 楼房重建](https://www.luogu.com.cn/problem/P4198)

## 多个 Tag 的线段树

普通的加减乘除和取模运算都可以使用 Lazy Tag 线段树维护。

但如果需要同时支持多种操作，则需要多个 Tag。

需要注意的要点是，你需要定义 Tag 的优先级。

例如，我们需要一棵同时支持区间加法和乘法的线段树：

定义 Tagp 为加法 Tag，Tagm 为乘法 Tag。

- 加法优先

`x = (x + tagp[x]) * tagm[x];`

先加$A$，再乘$B$。

```c++
tagp[x] = A;
tagm[x] = B;
```

先乘$A$，再加$B$。

```c++
tagm[x] = A;
push_down();
tagp[x] = B;
```

- 乘法优先

`x = (x * tagm[x]) + tagp[x];`

先加$A$，再乘$B$。

```c++
tagp[x] = A;
push_down();
tagm[x] = B;
```

先乘$A$，再加$B$。

```c++
tagm[x] = A;
tagp[x] = B;
```

### 练习题

[P3373 【模板】线段树 2](https://www.luogu.com.cn/problem/P3373)

## 维护区间开方操作

对于支持区间开方和区间求和的线段树，我们怎么做呢？🤔

经过思考，我们可以发现即使是很大的数，再经过很少次数的开方操作后，就能变成 1。

例如：$10^{18}$在经过 6 次开方并向下取整的操作后，就变成 1 了。

所以，我们记录当前区间的最大值，如果此值为 1 ，就无需再进行操作。

否则，我们进入子节点进行修改（一路到达叶子节点）。

听着比较暴力，但是时间复杂度是完全可接受的。

当然，如果把开方和其他操作（如加法）结合在一起，也是可以的。

```c++
void modify(int k, int l, int r, int L, int R) {
    if (L <= l && r <= R && mx[k] == 1) {
        return;
    if (l == r) {
        mx[k] = sqrt(c[k]);
        return;
    }
    push_down(k, l, r);
    int mid = (l + r) >> 1;
    if (L <= mid) modify(k << 1, l, mid, L, R);
    if (R > mid) modify(k << 1 | 1, mid + 1, r, L, R);
    push_up(k);
}
```

### 练习题

[Luogu P4145 上帝造题的七分钟2 / 花神游历各国](https://www.luogu.com.cn/problem/P4145)

[UOJ #228. 基础数据结构练习题](http://uoj.ac/problem/228)

## 线段树区间取 Min/Max

这里我们需要实现一种操作：对于一个区间$[l, r]$和一个数$x$，我们将其中每一个数$a[i]$修改为$min/max(a[i], x)$。

1. 数据满足单调性

我们可以利用二分求出需要修改的区间，再进行区间修改即可。

2. 数据无特殊性，支持区间加法和区间求极值。

我们记录两个 Tag，`add[x]`为节点 x 上增加的值，`mx[x]`为节点 x 上通过取 Max 得到的值。

我们令`add`标签的优先级更高。

那么$x$的真实值为`max(x + add[x], mx[x])`。

区间加$A$：max(x + add[x] + A, mx[x] + A)，更新`add[x]`和`mx[x]`标签。

区间取 Max $A$：max(x + add[x], max(mx[x], A))，更新`mx[x]`标签。

## 求区间$Mex$值

所谓$Mex$定义为区间中没有出现过的最小非负整数。

不需要支持修改，我们考虑离线算法，对所有的询问排序后处理。

假设对于所有以$l$为左端点的询问，我们设$mex[i]$为$[l, i]$区间的$Mex$值。显然，我们可以在$O(n)$时间内通过一个桶得出$mex[]$。

对于以$l + 1$为左端点的询问，我们考虑删除$a[l]$对$mex[]$的影响。

设$x = a[l]下一次出现的位置$，那么对于$mex[i >= x]$是不会有影响的。

对于$mex[i < x]$，如果$mex[i] > a[l]$，我们将$mex[i]$设为$a[l]$，即可从$l$转移到$l + 1$。

另外，由于$mex[]$是非降的，我们很简单地完成这个取 min 操作。

明显可以使用线段树在$log_{2}n$内完成。

### 练习题

[Luogu P4137 Rmq Problem / mex](https://www.luogu.com.cn/problem/P4137)

## 最大子段和

支持查询$[l, r]$区间内最大的连续子段的和。

我们考虑最大子段在当前节点中的分布:

- 最大子段在左/右儿子节点中

我们直接取最大值即可得出。

- 最大子段经过了中点，需要合并左右儿子节点的信息。

我们记录$res$为当前节点的最大子段和，$sum$为区间和，$prel$为从最左端开始的最大子段和，$prer$为以右端为结尾的最大子段和。

合并子节点信息时：

```c++
res[rt] = std::max(std::max(res[l], res[r]), prer[l] + prel[r])
sum[rt] = sum[l] + sum[r];
prel[rt] = std::max(prel[l], sum[l] + prel[r]);
prer[rt] = std::max(prer[r], sum[r] + prer[l]);
```

### 练习题

[Luogu SP1716 GSS3 - Can you answer these queries III](https://www.luogu.com.cn/problem/SP1716)

[Luogu P3582 [POI2015]KIN](https://www.luogu.com.cn/problem/P3582)

## 储存「上一次出现的位置（前驱）」

对于数组`a[]`，我们构造数组`b[]`，使得`b[i] =`值为`a[i]`的数在`a`数组中上一次出现的下标。

如何构造`b[]`？

- 用`std::map`

- 离散化 + 桶

如果我们维护当前节点内的前驱最大值，通过和当前节点的左边界做比较，即可得知此区间内是否有重复的数。

### 练习题

[Luogu P1972 [SDOI2009]HH的项链](https://www.luogu.com.cn/problem/P1972)

## 维护区间$gcd$

重点在于节点的合并。

细细思考一下，当前节点的$gcd$值，应该等于$gcd(tr[l].gcd, tr[r].gcd)$。

### 练习题

[Luogu P5278 算术天才⑨与等差数列](https://www.luogu.com.cn/problem/P5278)

## 将斜率储存在线段树中

笔者记得有这样一道题（实在是找不到原题了QAQ）

给出在一个二维平面内的若干矩形的坐标，（多组）询问如果从点$(0, 0)$发出一条以$k$为斜率的光线，是否会接触到矩形。

支持动态增加、删除矩形，不强制在线。

如果对于每组询问，我们遍历每一个矩形判断是否接触，显然是会超时的。

如何使用线段树求解呢？

对于每一个矩形，我们求出其以$(0, 0)$为起点的直线与其接触的斜率范围$[l, r]$。

将所有的斜率范围端点和查询是给出的斜率离散化后，在线段树中打上标记。

查询时单点查找是否有标记覆盖即可。

通过将斜率离散化后储存在线段树中，我们将这道题转化为区间修改、单点查询的题。

## 权值线段树

其实就是一棵普通的线段树，只不过我们维护的是以权值为下标的一个数列（也就是一个桶）的区间信息。

### 练习题

[Luogu P1486 [NOI2004]郁闷的出纳员](https://www.luogu.com.cn/problem/P1486)

## 权值线段树上的「二分」

假设我们需要知道权值线段树中第 k 个元素是什么。

在查询时，若我们所查的k小于等于左儿子的和，则进入左儿子查询第k大值。

若k大于左儿子的和，则进入右儿子查询第k - sum[l]大值。

```c++
int query(int k, int l, int r, int K) {
    if (l == r) return l;
    push_down(k, l, r);
    int mid = (l + r) >> 1;
    if (K <= sum[k << 1]) return query(k << 1, l, mid, K);
    else return query(k << 1 | 1, mid + 1, r, K - sum[k << 1]);
}
```

### 查询全局第 k 大

这其实很直观，我们只需把整个数列放进桶里，在查询时使用上述方法即可。

如果不需要支持修改，那么直接排序就好了。

如果需要支持区间修改，可以写树套树。

### 查询区间第 k 大

显然，对于区间$[l, r]$的查询，我们需要一棵区间$[l, r]$上的权值线段树。

假设对于当前询问，我们已经构建好了一棵这样的树，并获取了答案。

那么，对于下一个询问$[l', r']$，我们需要将当前左右端点$l, r$一个个挪到$l', r'$去，同时进行单点修改。

为了使挪动的操作尽可能少，我们使用莫队算法**离线**处理各个询问。

更简洁的做法是主席树（可持久化权值线段树）。

### 练习题

[POJ 2985](http://poj.org/problem?id=2985)

## 动态开点线段树

可以节省空间，或者在树套树、主席树中使用。

### 伪动态开点

先用一个`struct`来表示一个线段树节点，并提前开好一个大大的 buffer 数组和用来记录已分配节点个数的`cnt`变量。

在这里我们用`lch`和`rch`表示左右子节点，当然你也可以用指针。

```c++
struct node {
    int c, tag, lch, rch;
}buf[10000001];
int buf_cnt;
```

对于需要访问的新节点，我们调用`new_node`函数返回其在`buffer`数组中的下标。

```c++
int new_node() {
    return ++buf_cnt;
}
```

### 真动态开点

显然，上面的代码是提前开好了一个内存池（`buffer`数组），从内存池中分配节点。

我们也可以使用 C++ 的`new`和`delete`来动态申请、销毁`node`。

注意，此时用来表示左右子节点的只能是指针了。

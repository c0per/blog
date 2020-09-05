---
title: Manacher算法详解
date: 2018-03-13 18:12:13
tags:
- 算法讲解
- 字符串
- Manacher
categories:
- 字符串
- Manacher
---

# Manacher 最长回文子串算法详解

> 马拉车算法是非常快的求最长回文子串的算法

## 暴力算法

如何运用暴力算法求出最长回文子串的？

<!-- more -->

枚举回文串中点，向两边拓展，可以做到$O(n^2)$的时间复杂度。

## 改进方法

我们注意到，回文串是关于一个对称轴左右对称的，这个特性使得我们可以减少运算次数（蒟蒻不知道怎么减少=。=）。

## 算法思路

但有些回文串的长度为偶数，为了解决这个问题，我们可以在每个字符间隔插入一个分割字符，例如“#”。

```
    abcbe
==> #a#b#c#b#e#
```

显而易见的是，回文串在处理后依然为回文串，但所有回文串都变为了奇数长度。

接下来，Manacher算法定义了一个$Rl$数组，$Rl[i]$表示以字符$s[i]$为对称轴的最长回文串的半径。

例如：

```
#a#b#c#b#e#
```

其中c的最长回文串为“#b#c#b#”，那么，Rl[i]的值为4。

我们可以发现，$Rl-1$即为最长回文串的长度（处理前的）。

***

此外，Manacher算法引入了$pos$和$maxRight$两个变量，分别表示当前已处理的最长的回文串的长度和它的右端点（我储存的$maxRight$值加了$1$，即未被处理过的第一个的位置，为了便于实现）。

和暴力算法一样，我们循环i，回文串的对称轴，在这里，分为两种情况。

***

- $i​$在$maxRight​$的左端（$i \lt maxRight​$)

由于存在一个以$pos$为对称轴的，右端点为$maxRight-1$的回文串。所以，$Rl[i]$的值被初始化为$Rl[pos-(i-pos)]$。

当然，如果$i+Rl[i]>=maxRight$，即初始化后的i串的长度超过了$maxRight$，当然是不可行的。

所以，$Rl[i]=min(Rl[2*pos-i],maxRight-i)​$。

- $i​$在$maxRight的​$右端（$i>=maxRight​$)

没有办法通过以前的计算初始化一个较优的答案时，我们只能将Rl[i]初始化为1。

***

初始化后，我们像暴力算法一样向左右两边拓展，同时修改Rl值。

当无法继续拓展后，将$i+Rl[i]$与$maxRight$相比较。修改$maxRight$和$pos$。

***

如此可见，Manacher算法的复杂度为$O(2n)$，比暴力有不小的进步。

## 代码实现

```c++
void init()
{
    int tmp=strlen(s+1);
    for(int i=tmp;i>=1;--i)
    {
        s[i*2]=s[i];
        s[i*2+1]='#';
    }
    s[1]='#';
}
int manacher()
{
    int pos=0,maxright=1,i=1,ans=0;
    while(i<=len)
    {
        if(i<maxright)
            rl[i]=std::min(maxright-i,rl[2*pos-i]);
        else
            rl[i]=1;
        while(i-rl[i]>0&&i+rl[i]<=len&&s[i-rl[i]]==s[i+rl[i]])
            ++rl[i];
        if(i+rl[i]>maxright)
        {
            maxright=i+rl[i];
            pos=i;
        }
        ans=std::max(ans,rl[i]);
        ++i;
    }
    return ans;
}
int main()
{
    scanf("%s",s+1);
    init();
    printf("%d",manacher());
    return 0;
}
```

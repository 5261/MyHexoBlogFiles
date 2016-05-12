---
title: 烽火传递 - 单调队列优化DP
toc: true
date: 2016-04-04 17:41:21
tags:
  - OI
  - DP
  - 单调队列
  - NOIP
permalink: SignalFire-transmit
---

### 【题目描述】
烽火台又称烽燧，是重要的防御设施，一般建在险要处或交通要道上。一旦有敌情发生，白天燃烧柴草，通过浓烟表达信息：夜晚燃烧干柴，以火光传递军情。

在某两座城市之间有n个烽火台，每个烽火台发出信号都有一定的代价。
为了使情报准确的传递，在m个烽火台中至少要有一个发出信号。

现输入n、m和每个烽火台发出的信号的代价，请计算总共最少需要花费多少代价，才能使敌军来袭之时，情报能在这两座城市之间准确的传递。

### 【题目链接】
[Tyvj 1313](http://www.tyvj.cn/p/1313) 烽火传递 【NOIP2010】

<!--more-->

### 【解题思路】
动态规划，设$f[i]$表示在$[0, i]$中传递情报，且$i$烽火台必须参与传递的最小代价。

转移：
$$
f[i] =\\cases {
w_i & 0 ≤ i < m
\\\\ min\\{\\ f[k]\\ \\}\\ k \\in [i - m, i) & m ≤ i < n
}
$$
即考虑前一个烽火台的位置。

上式在计算$f[i]$时需要区间最小值，且决策区间是向右平移的 。

因此我们使用单调队列来优化。

考虑到要保证情报的传递，答案为：
$$
ans = max\\{\\ f[i]\\ \\}\\ i \\in [max(n - m, 0), n)
$$

### 【AC代码】
```c++
#include <cstdio>
#include <climits>
#include <deque>
#include <algorithm>

#define MAXN 1000000

template <typename Item> struct MonoQueue{
    std::deque<Item> data, aux;

    void push(const Item &x){
        data.push_back(x);
        while(!aux.empty() && aux.back() > x) aux.pop_back();
        aux.push_back(x);
    }

    void pop(){
        Item x = data.front();
        data.pop_front();
        if(x == aux.front()) aux.pop_front();
    }

    size_t size(){
        return data.size();
    }

    Item min(){
        return aux.front();
    }
};

int a[MAXN], f[MAXN];

int main(){
    int n, m;

    scanf("%d%d", &n, &m);
    for(int i = 0; i < n; i++) scanf("%d", a + i);

    MonoQueue<int> Q;
    for(int i = 0; i < m; i++) Q.push(f[i] = a[i]);
    for(int i = m; i < n; i++){
        if(Q.size() == m + 1) Q.pop();
        f[i] = Q.min() + a[i];
        Q.push(f[i]);
    }

    int ans = INT_MAX;
    for(int i = std::max(n - m, 0); i < n; i++) ans = std::min(ans, f[i]);
        
    printf("%d\n", ans);

    return 0;
}

```
就是这样啦。

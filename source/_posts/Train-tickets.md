---
title: 火车票 - 划分DP
toc: true
date: 2016-04-02 23:48:20
tags: 
  - OI
  - DP
  - 划分DP
permalink: Train-tickets
---

### 【题目描述】
铁路线上有n（2 ≤ n ≤ 10000）个火车站，每个火车站到该线路的首发火车站距离都是已知的。任意两站之间的票价如下所示：

当 $0 < x ≤ L\_1 ,\\ price = C\_1$
当 $L\_1 < x ≤ L\_2,\\ price = C\_2$
当 $L\_2 < x ≤ L\_3,\\ price = C\_3$

其中 $L\_1，L\_2，L\_3，C\_1，C\_2，C\_3$ 都是已知的正整数，
且$1≤L\_1 < L\_2 < L\_3 ≤ 10^9, 1 ≤ C\_1 < C\_2 < C\_3 ≤ 10^9$。
显然若两站之间的距离大于L3，那么从一站到另一站至少要买两张票。

注意：每一张票在使用时只能从一站开始到另一站结束。

对于给出的起点和终点，求出最省钱的方案。

### 【题目链接】
[Tyvj 3317](http://tyvj.cn/p/3317) 火车票（清新版题面）
[CodeVS 1349](http://codevs.cn/problem/1349/) 板猪的火车票 （恶搞版题面）

<!--more-->

### 【解题思路】

划分DP，设$f[i]$表示从起点$s$到$i$的最小花费。

转移：
$$
f[i] = min\\{\ f[k] + cost(k, i)\\ \\}\\ k \\in [s, i)\\ \\&\\& \\ dist(k, i) ≤ L3
$$
其中$dist(i, j)$表示$i$到$j$的距离，$cost(i, j)$表示$i$到$j$的车票价格，显然都可以$O(1)$计算。

转移思路很简单，就是枚举从火车站k到这儿来，取最小花费。

边界：
$$
f[s] = 0
$$

### 【AC代码】
因为心情好，于是就写记忆化搜索啦。~~（似乎并没有什么关联）~~
```c++
#include <cstdio>
#include <algorithm>
#include <climits>

#define MAXN 10000

int a[MAXN];

int n, s, t;
int L1, L2, L3, C1, C2, C3;

int f[MAXN];
bool visited[MAXN];

inline int cost(int i, int j){
    int dist = a[j] - a[i];
    if (dist > L3) return -1;
    else if (dist > L2) return C3;
    else if (dist > L1) return C2;
    else return C1;
}

int search(int v){
    int &ans = f[v];

    if(visited[v]) return ans;
    else visited[v] = true;

    if(v == s) ans = 0;
    else{
        ans = INT_MAX;
        for(int k = s; k < v; k++){
            int c = cost(k, v);

            if(c == -1) continue;
            else ans = std::min(ans, search(k) + c);
        }
    }

    return ans;
}

int main(){
    scanf("%d %d %d %d %d %d", &L1, &L2, &L3, &C1, &C2, &C3);
    scanf("%d%d%d", &n, &s, &t), s--, t--;
    for(int i = 1; i < n; i++) scanf("%d", a + i);

    printf("%d", search(t));
    return 0;
}

```
就是这样啦

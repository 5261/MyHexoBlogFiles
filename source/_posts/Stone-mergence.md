---
title: 石子归并 - 区间DP
toc: true
date: 2016-03-31 19:42:13
tags:
  - OI
  - DP
  - 区间DP
  - NOIP
  - 平衡树
  - GarsiaWachs算法
permalink: Stone-mergence
---

### 【题目描述】
有n堆石子排成一列，每堆石子有一个重量$w_i$, 每次合并可以合并相邻的两堆石子，一次合并的代价为两堆石子的重量和$w\_i + w\_{i + 1}$。
问安排怎样的合并顺序，能够使得总合并代价达到最小。

### 【题目链接】
[CodeVS 1048](http://codevs.cn/problem/1048/) 石子归并 【NOIP 2000】

<!--more-->

### 【解题思路】
区间DP入门题目，设$f[l][r]$表示将区间$[l, r]$合并成一堆的最小代价，则有：

$$ f[l][r] = min\\{\\ f[l][k] + f[k+1][r]\\ \\} + sum(l,\\ r),k \\in [l, r) $$

其中 $sum(l, r) = \\sum\_{i\\ =\\ l}^r w\_i$

即枚举区间$[l, r]$是由$[l, k]$与$[k + 1, r]$合并而来。

边界条件：
$$f[i][i] = 0$$

注意转移的顺序，既不是按左端点枚举也不是按右端点枚举。

考虑我们计算一个区间时需要哪些区间的信息——比当前区间长度短的区间！

故应按照区间长度由短到长计算，这也是区间型DP的一般特征。

时间、空间复杂度均为$O(n^2)$。

### 【AC代码】
```c++
#include <cstdio>
#include <climits>
#include <algorithm>

#define MAXN 100

int a[MAXN], n;
int f[MAXN][MAXN];

inline int sum(int l, int r){
    if(l == 0) return a[r];
    else return a[r] - a[l - 1];
}

int main(){
    scanf("%d", &n);
    for(int i = 0; i < n; i++){
        scanf("%d", a + i);
        if(i) a[i] += a[i - 1];
    }

    for(int i = 0; i < n; i++) f[i][i] = 0;
    for(int len = 1; len < n; len++){
        for(int l = 0; l < n - len; l++){
            int r = l + len;
            int min = INT_MAX;
            for(int k = l; k < r; k++){
                min = std::min(min, f[l][k] + f[k + 1][r]);
            }
            f[l][r] = min + sum(l, r);
        }
    }

    printf("%d\n", f[0][n - 1]);
    return 0;
}

```

### 【扩展：环形石子归并】
考虑一个小扩展，即石子不是排成一列而是成环形，该如何处理呢？
将环长度翻倍，展开成链，枚举起点，像上面一样计算即可。
时间复杂度$O(n^3)$。

### 【环形石子归并代码】
[CodeVS 2102](http://codevs.cn/problem/2102/) 石子归并2

```c++
#include <cstdio>
#include <climits>
#include <algorithm>
#include <cstring>

#define MAXN 100

int a[MAXN * 2], n;
int preSum[MAXN * 2];
int f[MAXN][MAXN], g[MAXN][MAXN];
int minSocre = INT_MAX, maxSocre = INT_MIN;

inline int sum(int l, int r, int start){
    return preSum[r + start] - preSum[l + start - 1];
}

inline void solve(int start){
    memset(f, 0, sizeof f), memset(g, 0, sizeof g);
    for(int len = 1; len < n; len++){
        for(int l = 0; l < n - len; l++){
            int r = l + len;
            int min = INT_MAX, max = INT_MIN;
            for(int k = l; k < r; k++){
                min = std::min(min, f[l][k] + f[k + 1][r]);
                max = std::max(max, g[l][k] + g[k + 1][r]);
            }
            f[l][r] = min + sum(l, r, start);
            g[l][r] = max + sum(l, r, start);
        }
    }

    minSocre = std::min(minSocre, f[0][n - 1]);
    maxSocre = std::max(maxSocre, g[0][n - 1]);
}

int main(){
    scanf("%d", &n);
    for(int i = 0; i < n; i++){
        scanf("%d", a + i);
        a[i + n] = a[i];
    }

    preSum[0] = a[0];
    for(int i = 1; i < 2 * n; i++) preSum[i] = preSum[i - 1] + a[i];

    for(int i = 0; i < n; i++) solve(i);

    printf("%d %d\n", minSocre, maxSocre);

    return 0;
}

```

### 【留坑】

石子归并还有名为GarsiaWachs的解法，借助平衡树可将时间复杂度优化到$O(nlogn)$。

然而留坑待填 ...

就先这样子啦。

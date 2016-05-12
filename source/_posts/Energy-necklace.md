---
title: 能量项链 - 区间DP
toc: true
date: 2016-04-01 21:15:47
tags:
  - OI
  - DP
  - 区间DP
  - NOIP

permalink: Energy-necklace
---

### 【题目描述】
在Mars星球上，每个Mars人都随身佩带着一串能量项链。在项链上有N颗能量珠。能量珠是一颗有头标记与尾标记的珠子，这些标记对应着某个正整数。并且，对于相邻的两颗珠子，前一颗珠子的尾标记一定等于后一颗珠子的头标记。因为只有这样，通过吸盘（吸盘是Mars人吸收能量的一种器官）的作用，这两颗珠子才能聚合成一颗珠子，同时释放出可以被吸盘吸收的能量。如果前一颗能量珠的头标记为$m$，尾标记为$r$，后一颗能量珠的头标记为$r$，尾标记为$n$，则聚合后释放的能量为$m \* r\* n$（Mars单位），新产生的珠子的头标记为$m$，尾标记为$n$。

需要时，Mars人就用吸盘夹住相邻的两颗珠子，通过聚合得到能量，直到项链上只剩下一颗珠子为止。显然，不同的聚合顺序得到的总能量是不同的，请你设计一个聚合顺序，使一串项链释放出的总能量最大。

### 【题目链接】
[CodeVS 1154](http://codevs.cn/problem/1154/) 能量项链 【NOIP 2006】

<!--more-->

### 【解题思路】
这是一个环，首先想到将其长度翻倍，展开成链，转化成区间上问题。

这样环以$i$为起点就对应了链上的区间$[i, i + n)$。

然后进行区间DP，设$f[l][r]$表示区间$[l, r]$上的珠子聚合后释放的最大能量。
用 $a\_i$ 表示第 $i$ 颗珠子的头标记（也是第 $i + 1$ 颗珠子的尾标记），转移如下。
$$
f[l][r] = max\\{f[l][k] + f[k + 1][r] + a\_i \* a\_{k + 1} \* a\_{j + 1}\\},k \\in [l, r)
$$

即枚举区间$[l,r]$是由区间$[l, k]$和$[k + 1， r]$合并而来，取最大值。

时间复杂度$O(n^3)$。

### 【AC代码】
采用记忆化搜索实现。
```c++
#include <cstdio>
#include <climits>
#include <algorithm>
#include <cstring>

#define MAXN 100

int a[(MAXN << 1) + 1];
int f[MAXN << 1][MAXN << 1];
bool visited[MAXN << 1][MAXN << 1];

int search(int l, int r){
    // [l, r]
    int &ans = f[l][r];

    if(visited[l][r]) return ans;
    else visited[l][r] = true;

    if(l == r) ans = 0;
    else if(r - l == 1) ans = a[l] * a[r] * a[r + 1];
    else{
        ans = INT_MIN;
        // [l, k] + [k + 1][r]
        for(int k = l; k < r; k++){
            ans = std::max(ans, search(l, k) + search(k + 1, r) + a[l] * a[k + 1] * a[r + 1]);
        }
    }

    return ans;
}

int main(){
    int n;
    scanf("%d", &n);
    for(int i = 0; i < n; i++) scanf("%d", a + i), a[i + n] = a[i];
    a[n << 1] = a[0]; // 处理访问最后一个珠子的尾标记时的越界情况

    search(0 , 2 * n - 1);

    int ans = INT_MIN;
    for(int i = 0; i < n; i++) ans = std::max(ans, f[i][i + n - 1]);

    printf("%d\n", ans);

    return 0;
}

```

就是这样啦。

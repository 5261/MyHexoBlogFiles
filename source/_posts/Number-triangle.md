---
title: 数字三角形 - DP
toc: true
date: 2016-03-26 21:01:56
tags:
  - OI
  - DP
  - 坐标型dp
  - IOI
permalink: Number-triangle
---

决定从头重新系统的学一下DP，夯实基础，嗯，从简单开始。

### 【题目描述】
有一数字三角形，从顶部出发，在每一结点可以选择向左走或者向右走，一直走到底层。
要求找出一条路径，使路径上的值的和最大。

<!--more-->

### 【题目链接】
[CodeVS 1220](http://codevs.cn/problem/1220/)

### 【解题思路】

用二维数组储存数字，记为$a$。
动态规划，设$f\_\{i, j\}$表示以$(i, j)$为起点所能获得的最大数字和。
转移:
$$
f\_\{i, j\} = max(f\_\{i + 1, j\}, f\_\{i + 1, j + 1\}) + a\_\{i, j\}
$$
边界：
$$
f\_\{n - 1, i\} = a\_\{n - 1, i\}
$$
计算顺序：
因为计算第$i$行时需要下一行i + 1的信息，故应倒序枚举行。

### 【AC代码】
```c++
#include <cstdio>
#include <algorithm>

#define MAXM 100
#define MAXN 100

int a[MAXM][MAXN];
int f[MAXM][MAXN];

int main(){
    int n;

    scanf("%d", &n);
    for(int i = 0; i < n; i++){
        for(int j = 0; j <= i; j++){
            scanf("%d", a[i] + j);
        }
    }

    for(int i = 0; i < n; i++) f[n - 1][i] = a[n - 1][i];
    for(int i = n - 2; i >= 0; i--){
        for(int j = 0; j <= i; j++){
            f[i][j] = std::max(f[i + 1][j], f[i + 1][j + 1]) + a[i][j];
        }
    }

    printf("%d\n", f[0][0]);
    return 0;
}

```
就这样啦
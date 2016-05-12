---
title: 编辑距离问题 - 线性DP
toc: true
date: 2016-04-03 21:17:43
tags: 
  - OI
  - DP
  - 线性DP
permalink: Edit-dist
---

### 【题目描述】

设A和B是2个字符串。要用最少的字符操作将字符串A转换为字符串B。这里所说的字符操作包括：

(1) 删除一个字符；

(2) 插入一个字符；

(3) 将一个字符改为另一个字符。

将字符串A变换为字符串B所用的最少字符操作数称为字符串A到B的编辑距离，记为d(A,B)。试编写程序，对任给的2个字符串A和B，计算出它们的编辑距离d(A,B)。

### 【题目链接】
[CodeVS 2598](http://codevs.cn/problem/2598/) 编辑距离问题

<!--more-->

### 【解题思路】
线性DP，设$f[i][j]$表示A的长度为$i$的前缀和B的长度为$j$的前缀之间的编辑距离，之所以设长度而不设下标，是因为涉及到空串的问题，用下标的话边界不好处理。

转移：
$$
f[i][j] = \\cases {
f[i - 1][j - 1] 
&  A[i - 1] = B[j - 1]
\\\\
min(f[i - 1][j], f[i][j - 1], f[i - 1][j - 1]) + 1
& A[i - 1] ≠ B[j - 1]
}
$$

当前两个字符相同时，无需操作，直接等于上一位。
不同时，有三种可选的操作
(1)编辑到$i, j - 1$然后删除A的最后一个字符。
(2)编辑到$i - 1, j$然后在A最后面增加一个字符。
(3)编辑到$i - 1, j - 1$然后修改最后一位使其和B相同。
取最小值。

边界，任何串到空串的编辑距离为该串本身的长度。
$$
f[0][0] = 0
$$

$$
f[i][0] = i
$$

$$
f[0][j] = j
$$

### 【AC代码】
采用递推决策实现，注意下标。
```c++
#include <cstdio>
#include <algorithm>
#include <cstring>

#define MAXN 4000

char A[MAXN + 1], B[MAXN + 1];
int f[MAXN + 1][MAXN + 1];

inline int min(const int &a, const int &b, const int &c){
    return std::min(std::min(a, b), c);
}

int main(){
    scanf("%s\n%s", A, B);
    int n = strlen(A), m = strlen(B);

    f[0][0] = 0;
    for(int i = 1; i <= n; i++) f[i][0] = i;
    for(int j = 1; j <= m; j++) f[0][j] = j;

    for(int i = 1; i <= n; i++) for(int j = 1; j <= m; j++){
        int &ans = f[i][j];
        if(A[i - 1] == B[j - 1]) ans = f[i - 1][j - 1];
        else ans = min(f[i - 1][j], f[i][j - 1], f[i - 1][j - 1]) + 1;
    }
    printf("%d\n", f[n][m]);
    return 0;
}

```
就是这样啦

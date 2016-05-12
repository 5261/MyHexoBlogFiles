---
title: 括号序列 - 区间DP
toc: true
date: 2016-03-31 20:04:32
tags: 
  - OI
  - DP
  - 区间DP 
permalink: Brackets-list
---
### 【题目描述】
我们用以下规则定义一个合法的括号序列：

（1）空序列是合法的
（2）假如S是一个合法的序列，则 (S) 和[S]都是合法的
（3）假如A 和 B 都是合法的，那么AB和BA也是合法的

例如以下是合法的括号序列：

(), [], (()), ([]), ()[], ()[()]

以下是不合法括号序列：

(, [, ], )(, ([]), ([()

 现在给定一些由'(', ')', '[', ，']'构成的序列 ，请添加尽量少的括号，得到一个合法的括号序列。
 
### 【题目链接】
[CodeVS 3657](http://codevs.cn/problem/3657/) 括号序列

<!--more-->

### 【解题思路】
经典区间DP，设$f[l][r]$表示将序列$[l, r]$补成合法括号序列最少的括号数。

转移如下，用$S$代表原序列：
$$f[l][r] = min:
\\\\ f[l + 1][r - 1],when\\ S\_l\\ can\\ match\\ with\\ S\_r,
\\\\ min\\{\\ f[l][k] + f[k + 1][r]\\ \\},k \\in [l, r)$$
即分别考虑合法括号序列定义的规则2和规则3，取最小值。

边界：
$$f[i][i] = 1$$
（单独一个括号是不合法的，需要补一个括号）

依旧按照区间长度从短到长计算。

### 【AC代码】

```c++
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <climits>

#define MAXN 100

char S[MAXN + 1];
int f[MAXN][MAXN];

int main(){
    scanf("%s", S);
    int n = strlen(S);

    for(int i = 0; i < n; i++) f[i][i] = 1;
    for(int len = 1; len < n; len++){
        for(int l = 0; l < n - len; l++){
            int r = l + len;

            int &ans = f[l][r];
            ans = INT_MAX;

            if((S[l] == '(' && S[r] == ')')
            || (S[l] == '[' && S[r] == ']')
            ) ans = std::min(ans, f[l + 1][r - 1]);
            
            for(int k = l; k < r; k++){
                ans = std::min(ans, f[l][k] + f[k + 1][r]);
            }
        }
    }

    printf("%d\n", f[0][n - 1]);
    return 0;
}

```

就是这样啦。

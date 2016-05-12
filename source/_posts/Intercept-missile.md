---
title: 拦截导弹 - 最长不上升子序列
toc: true
date: 2016-03-26 21:44:26
tags:
  - OI
  - DP
  - 线性DP
  - Dilworth定理
  - LIS
  - NOIP
permalink: Intercept-missile
---

### 【题目描述】

某国为了防御敌国的导弹袭击，发展出一种导弹拦截系统。但是这种导弹拦截系统有一个缺陷：虽然它的第一发炮弹能够到达任意的高度，但是以后每一发炮弹都不能高于前一发的高度。某天，雷达捕捉到敌国的导弹来袭。由于该系统还在试用阶段，所以只有一套系统，因此有可能不能拦截所有的导弹。

求这套系统最多能拦截多少导弹，以及如果要拦截所有导弹最少要配备多少套这种导弹拦截系统。

<!--more-->

### 【题目链接】
[CodeVS 1044](http://codevs.cn/problem/1044/)

### 【解题思路】
经典线性模型，最长不上升子序列。
动态规划，设$f\_\{i\}$表示以$i$为起点的最长不上升子序列的长度。
转移：
$$f\_\{i\} = max\\{\\ f\_\{j\}\\ \\} + 1,\\\\ i < j < n\\ \\&\\&\\ a\_j \\leq\\  a\_i$$
边界:
$$f\_\{n - 1\} = 1$$
计算顺序:
因为计算$f_i$时需要$i$之后的所有信息，故应倒序枚举$i$。

答案：
$$ans = max\\{f\_i\\},\\ 0 \\leq i < n$$

对于第二问，实际上是求其不上升子序列的最小划分，根据$Dilworth$定理：

链的最少划分数 = 反链的最长长度。

即第二问是求其最长上升子序列的长度，方法类似。

### 【AC代码】

```c++
#include <cstdio>
#include <algorithm>

#define MAXN 20

int a[MAXN], f[MAXN], g[MAXN];

int main(){
    int n = 0;

    while(scanf("%d", a + n) == 1) n++;

    f[n - 1] = g[n - 1] = 1;
    for(int i = n - 2; i >= 0; i--){
        f[i] = g[i] = 1;
        for(int j = i + 1; j < n; j++){
            if(a[j] <= a[i] && f[j] + 1 > f[i]){
                f[i] = f[j] + 1;
            }
            if(a[j] > a[i] && g[j] + 1 > g[i]){
                g[i] = g[j] + 1;
            }
        }
    }

    int x = 0, y = 0;
    for(int i = 0; i < n; i++){
        x = std::max(x, f[i]);
        y = std::max(y, g[i]);
    }
    printf("%d\n%d", x, y);

    return 0;
}

```
就是这样啦
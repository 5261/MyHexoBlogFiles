---
title: 最大连续子序列和
toc: true
date: 2016-03-26 23:08:01
tags:
  - OI
  - DP
  - 线性DP
permalink: Max-sum-of-Continuous-sub-sequence
---

### 【问题描述】

给定$n$个数 $a\_1 , a\_2 , ... , a\_n$

定义 $$f(i,j) = \\sum\_\{k = i\}^\{j\} a\_k$$

求 $f(i,j)$ 的最大值

<!--more-->

### 【题目链接】
[CodeVS 3155](http://codevs.cn/problem/3155/)

### 【解题思路】
经典问题，最大连续子序列和，有多种解法，是讲解时间复杂度相关的绝好例题(->_->)。

基于动态规划的$O(n)$做法：
设$f\_i$表示以$i$为终点的最大连续子序列和，则：
$$f\_i = max(f\_i + a\_i,\\ a\_i)
\\\\ \\ \\  = max(f\_i, 0) +a\_i$$
即考虑选不选$a\_i$这个元素，选的话就加上，不选的话就从头来过。

答案为所有$f\_i$最大值。

可以利用滚动数组优化空间复杂度到$O(1)$。

### 【AC代码】
```c++
#include <cstdio>
#include <algorithm>

int main(){
    int n;
    scanf("%d", &n);

    int sum = 0, ans = 0;
    for(int i = 0; i < n; i++){
        int x;
        scanf("%d", &x);
        sum = std::max(sum, 0) + x;
        ans = std::max(ans, sum);
    }

    printf("%d\n", ans);
    return 0;
}

```
就是这样啦

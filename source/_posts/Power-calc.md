---
title: 幂运算 - 迭代加深搜索
toc: true
date: 2016-03-14 07:21:03
tags:
  - OI
  - 搜索
  - 迭代加深搜索
  - IDAstar
  - 剪枝
permalink: Power-calc
---

### 【题目链接】

[CodeVS 2541](http://www.codevs.cn/problem/2541/) 幂运算

### 【题目描述】
从m开始，我们只需要6次运算就可以计算出$m^{31}$：

$m^2=m×m，m^4=m^2×m^2，m^8=m^4×m^4$，
$m^{16}=m^8×m^8，m^{32}=m^{16}×m^{16}，m^{31}=m^{32}÷m $。
    
请你找出从$m$开始，计算$m^n$的最少运算次数。在运算的每一步，都应该是$m$的正整数次方，换句话说，类似$m^{-3}$是不允许出现的。
(1<=n<=1000)

<!--more-->

### 【解题思路】

这题比较简单，以已经得到的指数集合为状态，扩展操作是在其中任选两个数做加减法，且不要产生重复的数（产生重复的数显然不合算）。

考虑简单剪枝，如果记当前的得到的最大指数$max$， 当$max \* 2^{limit - d} < n$时剪枝，含义就是一直用最大的数做加法，到达深度上限时指数也到不了$n$，这实际上也是IDA\*算法，只不过乐观估价函数为对数式，为方便计算将不等式变形为指数式。

另外的，我们有一个未能证明的优化，每次总是使用"刚刚得到"的那个指数，这个优化的正确性未能证明，但1000以内不存在反例。

### 【AC代码】
```c++
#include <cstdio>
#include <algorithm>

#define MAXN 1000

int n;
int exp[MAXN];  // 指数集合

int limit;

bool dfs(int d, int max){
    if(exp[d] == n) return true;
    if(d == limit) return false;

    if((max << limit - d) < n) return false; // 剪枝

    for(int i = d; i >= 0; i--){
        exp[d + 1] = exp[d] + exp[i]; // 考虑加法
        if(dfs(d + 1, std::max(exp[d], exp[d + 1]))) return true;

        exp[d + 1] = exp[d] - exp[i]; // 考虑减法
        if(dfs(d + 1, std::max(exp[d], exp[d + 1]))) return true;
    }
    // 这里都使用了数字exp[d]，即刚刚得到的数字。
    return false;
}

inline int solve(){
    if(n == 1) return 0;
    exp[0] = 1;
    for(limit = 1; limit < MAXN; limit++){
        if(dfs(0, 1)) return limit;
    }
}

int main(){
    scanf("%d", &n);
    printf("%d", solve());
    return 0;
}
```

就这样啦
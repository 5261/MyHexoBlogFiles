---
title: 数字游戏 - 划分DP
toc: true
date: 2016-04-03 16:42:38
tags:
  - OI
  - DP
  - 划分DP
  - NOIP
permalink: Number-game
---

### 【题目描述】
丁丁最近沉迷于一个数字游戏之中。这个游戏看似简单，但丁丁在研究了许多天之后却发觉原来在简单的规则下想要赢得这个游戏并不那么容易。游戏是这样的，在你面前有一圈整数（一共n个），你要按顺序将其分为m个部分，各部分内的数字相加，相加所得的m个结果对10取模后再相乘，最终得到一个数k。游戏的要求是使你所得的k最大或者最小。

例如，对于下面这圈数字（n=4，m=2）：
```
                                  2

                   4                           -1

                                 3
```
当要求最小值时，((2-1) mod 10)×((4+3) mod 10)=1×7=7，要求最大值时，为((2+4+3) mod 10)×(-1 mod 10)=9×9=81。特别值得注意的是，无论是负数还是正数，对10取模的结果均为非负值。

丁丁请你编写程序帮他赢得这个游戏。

### 【题目链接】
[CodeVS 1058](http://codevs.cn/problem/1085/) 数字游戏 【NOIP 2003】

<!--more-->

### 【解题思路】
这是一个环，然而因为要做划分DP而不是区间DP，所以并不能用将环倍长展开的方法来降低时间复杂度了。

所以我们只能考虑枚举起点展开成链，分别做划分DP。

以最小值为例，设$f[i][j]$表示区间$[0, i]$内划分成$j$段的最小得分。

转移：
$$
f[i][j] = min\\{\\ f[k][j - 1] \* sumMod(k + 1, i) \\}\\ k \\in [j - 1, i)
$$
其中$sumMod(i, j)$表示区间$[i, j]$内数字和对$10$取模后的结果。

即
$$
sumMod(l, r) = \\sum\_{i = l}^r a\_i \\pmod {10}
$$

就是枚举划分点的位置，依然要注意给前面的划分留住足够位置。

边界：
$$
f[i][1] = sumMod(0, i)
$$

### 【取余和取模】

注意取余和取模的区别，对于两个整数$a, b$，计算取模运算或者求余运算的方法都是：
1. 求整数商 $c = a / b$
2. 求余数或模数$r = a - c \* b$
区别在于第一步的除法计算，取模运算在这一步是向下取整的，而取余运算在这一步是向0取整的。

因此，当a和b符号一致时，求模运算和求余运算所得结果一致。但是当符号不一致的时候，结果不一样。求模运算结果的符号和b一致，求余运算结果的符号和a一致。

不同的语言中`%`表示的含义不同，像`C/C++`中`%`表示的是取余，而像`Python`中`%`表示的是取模，这实际上反映的是取整方法的差异。

而本题中要求取模，用取余表示取模的正确姿势是：
```c++
(X % P + P) % P
```
权且当作科普。

### 【AC代码】
```c++
#include <cstdio>
#include <algorithm>
#include <climits>

#define MAXN 50
#define MAXM 9
#define P 10

int a[MAXN], preSum[MAXN];
int f[MAXN][MAXM + 1], g[MAXN][MAXM + 1];
int n, m;
int minAns, maxAns;

inline int sumMod(int l, int r){
    return (( l ? preSum[r] - preSum[l - 1] : preSum[r] ) % P + P) % P;
}

inline void work(){
    for(int i = 0; i < n; i++) f[i][1] = g[i][1] = sumMod(0, i);
    for(int i = 0; i < n; i++){
        for(int j = 2; j <= m; j++){
            int &min = f[i][j], &max = g[i][j];

            min = INT_MAX, max = INT_MIN;
            for(int k = j - 1; k < i; k++){
                min = std::min(min, f[k][j - 1] * sumMod(k + 1, i));
                max = std::max(max, g[k][j - 1] * sumMod(k + 1, i));
            }
        }
    }

    minAns = std::min(minAns, f[n - 1][m]);
    maxAns = std::max(maxAns, g[n - 1][m]);
}

inline void revolve(){
    int x = a[0];
    for(int i = 0; i < n - 1; i++) a[i] = a[i + 1];
    a[n - 1] = x;
    
    preSum[0] = a[0];
    for(int i = 1; i < n; i++) preSum[i] = preSum[i - 1] + a[i];
}

int main(){
    scanf("%d%d", &n, &m);
    for(int i = 0; i < n; i++) scanf("%d", a + i);

    minAns = INT_MAX, maxAns = INT_MIN;
    for(int i = 0; i < n; i++){
        revolve();
        work();
    }

    printf("%d\n%d", minAns, maxAns);

    return 0;
}

```
就是这样啦
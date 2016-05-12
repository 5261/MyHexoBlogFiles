---
title: windy数 - 数位DP
toc: true
tags:
  - OI
  - DP
  - 数位DP
permalink: Windy-number
date: 2016-04-28 21:17:55
---

#### 【题目描述】
windy定义了一种windy数。不含前导零且相邻两个数字之差至少为2的正整数被称为windy数。 

windy想知道，在A和B之间，包括A和B，总共有多少个windy数？

#### 【题目链接】
[BZOJ 1026](http://www.lydsy.com/JudgeOnline/problem.php?id=1026) windy数

<!--more-->

#### 【解题思路】

数位DP入门...

第一步预处理。

设 $ f[i][j] $表示开头数字为 $ j $ 的 $ i $ 位的windy数的个数。

枚一下后一位的数字$ k $，保证满足windy数条件时才转移，这样递推一下就好，大概这样：

$$ f[i][j] = \sum_{k = 0}^9 f[i - 1][k] * [\lvert j - k \rvert >= 2]$$

其中$ [exp] $，当表达式$exp$为真时值为0，否则为1。

在这儿，通俗的说就是表示只有差不小于2时才能算进去啦。

第二步统计答案。

常见思路，区间$ [a, b] $的答案可以转化为区间$ [0, b] $的答案减去区间$ [0, a) $的答案。

而$ [0, x) $的答案，可以通过对$ x$逐位考虑得到。

比$ x $小的数，要么长度上比$ x $短，要么与$ x $长度相同，但在最长公共前缀的下一位比$ x $小。

长度比$ x $短的部分可以直接算进答案里。

而如果确定了某一位比$ x $小，则之后的位可以任取，个数同样可以由预处理的部分直接得到。

这就是数位DP的基本思想。

还可以借助 ”模板“ 的思想去理解，可参见刘汝佳《算法竞赛入门经典 - 训练指南》P114 图 2-8。

具体到本题，结合代码及注释理解一下：

```c++
inline int calc(int x){
    // 计算区间[0, x)的答案
    int d[MAXD];
    int n = split(x, d); // 表示将x的各位数字分离存入d数组中，同时返回长度n
    int ans = 0;
 
    for(int i = 1; i < n; i++) for(int j = 1; j <= 9; j++) ans += f[i][j]; 
    // 该行计入了所有长度小于n的数的答案，因为要求不含前导零，所以首位数字应j从1开始枚。
    
    for(int i = 1; i < d[0]; i++) ans += f[n][i];
    // 同样，因为前导零的缘故，要把最高位拿出来特殊处理
 
    for(int i = 1; i < n; i++){
        // 枚举到第i位时,d[0, i)是确定的前缀。
        int k = n - i; // 未确定的长度
        for(int j = 0; j < d[i]; j++) if(abs(d[i - 1] - j) >= 2) ans += f[k][j];
        // 因为前i - 1与x相同，所以枚举第i位数字j，只要j小于x的第i位（且满足windy数条件），后面的位就可以任取，并计入答案。
        if(abs(d[i - 1] - d[i]) < 2) break;
        // 此时前缀以不符合windy数条件，可以（而且应该）跳出。
    }
 
    return ans;
}
```

#### 【AC代码】
```c++
#include <cstdio>
#include <algorithm>
 
const int MAXD = 10;
 
int f[MAXD + 1][10];
 
inline void init(){
    for(int i = 0; i <= 9; i++) f[1][i] = 1;
    for(int i = 2; i <= MAXD; i++){
        for(int j = 0; j <= 9; j++){
            for(int k = 0; k <= 9; k++) if(abs(j - k) >= 2){
                f[i][j] += f[i - 1][k];
            }
        }
    }
}
 
inline int split(int x, int d[]){
    int n = 0;
    while(x) d[n++] = x % 10, x /= 10;
    for(int i = 0; i < n >> 1; i++) std::swap(d[i], d[n - i - 1]);
    return n;
}
 
inline int calc(int x){
    int d[MAXD];
    int n = split(x, d);
    int ans = 0;
 
    for(int i = 1; i < n; i++) for(int j = 1; j <= 9; j++) ans += f[i][j]; 
    for(int i = 1; i < d[0]; i++) ans += f[n][i];

    for(int i = 1; i < n; i++){
        int k = n - i;
        for(int j = 0; j < d[i]; j++) if(abs(d[i - 1] - j) >= 2) ans += f[k][j];
        if(abs(d[i - 1] - d[i]) < 2) break;
    }
 
    return ans;
}
 
inline int solve(int a, int b){
    return calc(b + 1) - calc(a);
}
 
int main(){
    init();
 
    int a, b;
 
    scanf("%d%d", &a, &b);
 
    printf("%d\n", solve(a, b));
 
    return 0;
}
```
就是这样啦。

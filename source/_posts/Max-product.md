---
title: 乘积最大 - 划分DP
toc: true
date: 2016-04-01 22:49:41
tags: 
  - OI
  - DP
  - 划分DP
  - NOIP
permalink: Max-product
---

### 【题目描述】
今年是国际数学联盟确定的“2000——世界数学年”，又恰逢我国著名数学家华罗庚先生诞辰90周年。在华罗庚先生的家乡江苏金坛，组织了一场别开生面的数学智力竞赛的活动，你的一个好朋友XZ也有幸得以参加。活动中，主持人给所有参加活动的选手出了这样一道题目：

设有一个长度为N的数字串，要求选手使用K个乘号将它分成K+1个部分，找出一种分法，使得这K+1个部分的乘积能够为最大。

同时，为了帮助选手能够正确理解题意，主持人还举了如下的一个例子：

有一个数字串：312， 当N=3，K=1时会有以下两种分法：

1) 3 \* 12=36
2) 31 \* 2=62

这时，符合题目要求的结果是：31*2=62

现在，请你帮助你的好朋友XZ设计一个程序，求得正确的答案。

### 【题目链接】
[CodeVS 1017](http://codevs.cn/problem/1017/) 乘积最大 【NOIP 2000】

<!--more-->

### 【解题思路】

动态规划，设$f[i][j]$表示在数字串的区间$[0, i]$中放置了$j$个乘号所得的最大乘积。

转移，其中$a[l][r]$表示数字串中区间$[l, r]$形成的数字。
$$
f[i][j] = max\\{\\ f[i][k - 1] \* a[i + 1][n]\\ \\}, k \\in [j - 1, n)
$$
即枚举位置$k$，尝试将乘号放在该位置的后面，注意枚举的下界，因为还要给前面$j - 1$个乘号留出足够的位置。

因为本题较老，数据实际上也比较小，使用`long long`即可通过。

### 【AC代码]

这里采用记忆化搜索来实现。

```c++
#include <cstdio>
#include <algorithm>

#define MAXN 40
#define MAXK 6

#define int64 long long

int n, k;
char num[MAXN];
int64 a[MAXN][MAXN], f[MAXN][MAXK + 1];
bool visited[MAXN][MAXK + 1];

inline void preTreat(){
    for(int i = 0; i < n; i++){
        for(int j = i; j < n; j++){
            a[i][j] = (j ? a[i][j - 1] * 10 : 0) + num[j] - '0';
        }
    }
}

int64 search(int n, int k){
    int64 &ans = f[n][k];
    
    if(k == 0) return a[0][n];
    else if(!visited[n][k]){
        visited[n][k] = true;

        for(int i = k - 1; i < n; i++){
            ans = std::max(ans, search(i, k - 1) * a[i + 1][n]);
        }
    }

    return ans;
}

int main(){
    scanf("%d%d%s", &n, &k, num);
    preTreat();
    printf("%lld\n", search(n - 1, k));
    return 0;
}
```
就是这样啦。
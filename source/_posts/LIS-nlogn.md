---
title: 最长上升子序列 - 树状数组 - O(nlogn)
toc: false
date: 2016-03-27 21:13:08
tags: 
  - OI
  - DP
  - 线性DP
  - LIS
  - 树状数组
permalink: LIS-nlogn
---

最长上升子序列的$O(nlogn)$做法。

<!--more-->

之前用到的较朴素的做法复杂度是$O(n^2)$的。

算法瓶颈在于每次转移时都要枚举获取最大值，我们考虑离散化整个序列，计算$f\_i$时相当于从$1 $到$ a\_i - 1$中选择一个$f$值最大的转移到$f\_i$。

前缀最大值 —— 这正是树状数组的拿手好戏！

于是转移复杂度降到$O(logn)$，整个算法复杂度降到$O(nlogn)$。

### 【代码】
[COGS 1398](http://cogs.top/cogs/problem/problem.php?pid=1398) 最长上升子序列
```c++
#include <cstdio>
#include <algorithm>
#include <climits>
#include <cstring>

#define MAXN 5000

int a[MAXN + 1];

int max[MAXN + 1];
int n;

int lowbit(signed int x){
    return x & -x;
}

int query(int i){
    int ans = INT_MIN;
    while(i){
        ans = std::max(ans, max[i]);
        i -= lowbit(i);
    }
    return ans;
}

void set(int i, int x){
    while(i <= n){
        max[i] = std::max(max[i], x);
        i += lowbit(i);
    }
} // 严格来说不能算修改，只能算设初值

int main(){
    // freopen("lis1.in", "r", stdin), freopen("lis1.out", "w", stdout);

    scanf("%d", &n);
    for(int i = 0; i < n; i++) scanf("%d", a + i);

    static int aux[MAXN + 1];
    memcpy(aux, a, sizeof a);
    std::sort(aux, aux + n);
    int size = std::unique(aux, aux + n) - aux;
    for(int i = 0; i < n; i++){
        a[i] = std::lower_bound(aux, aux + size, a[i]) - aux + 1;
    }

    int ans = INT_MIN;
    for(int i = 0; i < n; i++){
        int x;
        if(a[i] == 1) x = 1; // 注意边界
        else x = query(a[i] - 1) + 1;
        ans = std::max(ans, x);
        set(a[i], x);
    }

    printf("%d\n", ans);

    // fclose(stdin), fclose(stdout);
    return 0;
}

```
就是这样啦

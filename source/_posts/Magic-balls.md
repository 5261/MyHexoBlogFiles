---
title: Magic balls - LIS扩展
toc: true
date: 2016-03-28 20:55:50
tags:
  - OI
  - DP
  - 线性DP
  - 树状数组
  - LIS
permalink: Magic-balls
---

### 【题目链接】
[HDU 5125](http://acm.hdu.edu.cn/showproblem.php?pid=5125) Magic balls [BestCoder Round#20]

### 【题目描述】
原文是英文就不搬了，快捷版题意：

有$n$个数，每个数有两个属性$a_i$，$b_i$，每次操作可以交换一个数的两个属性，即选取一个$i$然后$swap(a_i, b_i)$，可以进行$k$次操作，问操作后$a_i$形成的最长上升子序列长度。

<!--more-->

### 【解题思路】

LIS问题的简单扩展，我们需要将交换操作也纳入到状态的表示中。

设 $dp0[i][j]$ 表示以 $i$ 结尾，已经用了 $j$ 次操作，并且第 $i$ 个元素并**没进行交换**的LIS长度。
设 $dp1[i][j]$ 表示以 $i$ 结尾，已经用了 $j$ 次操作，并且第 $i$ 个元素并**进行了交换**的LIS长度。

转移：
$$
dp0[i][j] = max\\{\\ 
dp0[t][j] \\ (a_t < a_i),\\ \\ \\ \\ 
dp1[t][j]\\ (a_t < b_i)
\\ \\} + 1$$

$$
dp1[i][j] = max\\{\\ 
dp0[t][j - 1] \\ (a_t < b_i),\\ \\ \\ \\ 
dp1[t][j - 1]\\ (b_t < b_i)
\\ \\} + 1
$$

答案显然是取所有方案最大值。

然而直接DP是过不了本题的，所以我们需要借助树状数组来加速。

像一般LIS一样，首先离散化，然后只需要建立$k$棵树状数组维护前缀最大值即可。

### 【AC代码】
。
```c++
#include <cstdio>
#include <cstring>
#include <algorithm>

#define MAXN 2002
#define MAXM 2002

int a[MAXN], b[MAXN];
int n, m, size;

struct TreeArray{
    int max[MAXN];

    void clear(){
        memset(max, 0, sizeof max);
    }

    int lowbit(signed int x){
        return x & -x;
    }

    int query(int i){
        int ans = 0;
        while(i){
            ans = std::max(ans, max[i]);
            i -= lowbit(i);
        }
        return ans;
    }

    void set(int i, int x){
        while(i <= size){
            max[i] = std::max(max[i], x);
            i += lowbit(i);
        }
    }

} arrays[MAXM];

inline int solve(){
    int ans = 0;

    for(int i = 0; i < n; i++){
        for(int j = std::min(i + 1, m); j >= 0; j--){
            int x = 0, y = 0;

            x = arrays[j].query(a[i] - 1) + 1; 
            if(j) y = arrays[j - 1].query(b[i] - 1) + 1;

            arrays[j].set(a[i], x), arrays[j].set(b[i], y);

            ans = std::max(ans, std::max(x, y));
        }
    }

    return ans;
}

int main(){
    int t;

    scanf("%d", &t);
    while(t--){
        scanf("%d%d", &n, &m);
        for(int i = 0; i < n; i++) scanf("%d%d", a + i, b + i);

        static int aux[MAXN << 1];

        memcpy(aux, a, sizeof a), memcpy(aux + n, b, sizeof b);

        std::sort(aux, aux + 2 * n);
        size = std::unique(aux, aux + 2 * n) - aux;

        for(int i = 0; i < n; i++){
            a[i] = std::lower_bound(aux, aux + size, a[i]) - aux + 1;
            b[i] = std::lower_bound(aux, aux + size, b[i]) - aux + 1;
        }

        for(int i = 0; i <= m; i++) arrays[i].clear();

        printf("%d\\n", solve());
    }

    return 0;
}

```
就是这样啦

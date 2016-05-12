---
toc: true
tags:
  - OI
  - DP
  - 斜率优化
  - BZOJ
  - SDOI
permalink: Journey-SDOI2016
date: 2016-04-22 19:14:36
title: 征途 - 斜率优化
---
#### 【题目描述】
Pine开始了从S地到T地的征途。
从S地到T地的路可以划分成n段，相邻两段路的分界点设有休息站。
Pine计划用m天到达T地。除第m天外，每一天晚上Pine都必须在休息站过夜。所以，一段路必须在同一天中走完。
Pine希望每一天走的路长度尽可能相近，所以他希望每一天走的路的长度的方差尽可能小。
帮助Pine求出最小方差是多少。
设方差是$ v $，可以证明，$ vm^2 $是一个整数。为了避免精度误差，输出结果时输出$ vm^2 $。

#### 【题目链接】
[BZOJ 4518](http://www.lydsy.com/JudgeOnline/problem.php?id=4518) 征途 【SDOI 2016】

<!--more-->

#### 【解题思路】
首先简单化下式子可得，
$$ 
\begin{align\*}
v 
&= \frac 1 m \sum (x\_i - \bar x )^2 \\\\
&= \frac 1 m \sum x\_i^2 - \bar x^2
\end{align\*}
$$
（注意那个$\bar x^2$是在求和外面的哈）

把平方项展开下就可以得到这样的式子，注意到后面$ \bar x^2 $一项是固定的，因此要想最小化方差只需最小化平方和就可以了。

我们设$ f[i][j] $表示前$ i $天，走到$ j $位置的最小平方和，另以$ s\_i $表示从原点到$ i $的距离，考虑是从前一天的$ k $位置走过来，转移显而易见，
$$ f[i][j] = \min\\{\ f[i - 1][k] + (s\_i - s\_j)^2, \ k \in [0, i] \\}  $$

直接做是 $ O(n^3) $的，可以过60分。

而正解就是斜率优化啦。

展开平方项，将$ k\_1 < k\_2$时，$ k\_2 $优于$ k\_1 $的充要条件写成斜率形式为，

$$ Slope(k\_1, k\_2) = \frac {g(k\_1) - g(k\_2)} {s\_{k\_1} - s\_{k\_2}} < 2s\_j$$

其中，

$$ g(k) = f[i - 1][k] + s\_k^2 $$

那么借助一个队列维护一系列有价值的决策，使得任意$ Slope(k\_t, k\_{t+1}) > 2s\_j$，故最优决策在队首。

另维护斜率单调减。

#### 【AC代码】
```c++
#include <cstdio>
#include <climits>
#include <algorithm>
 
const int MAXN = 3000;
const int MAXM = 3000;
 
typedef long long int64;
 
int n, m;
int64 sum[MAXN + 1];
 
inline int64 sqr(int64 x){
    return x * x;
}
 
inline void updateMin(int64 &x, int64 y){
    x = std::min(x, y);
}
 
inline int64 g(int64 *f, int k){
    return f[k] + sqr(sum[k]);
}
 
inline double slope(int64 *f, int i, int j){
    return (double)(g(f, i) - g(f, j)) / (sum[i] - sum[j]);
}
 
int64 f[MAXM + 1][MAXN + 1];
inline void dp(){
    for(int j = 0; j <= n; j++) f[1][j] = sqr(sum[j]);
 
    static int Q[MAXN + 1];
    for(int i = 2; i <= m; i++){
        int l = 0, r = -1;
 
        for(int j = 0; j <= n; j++){
            while(l < r && slope(f[i - 1], Q[r - 1], Q[r]) > slope(f[i - 1], Q[r], j)) r--;
            Q[++r] = j;
            while(l < r && slope(f[i - 1], Q[l], Q[l + 1]) < 2 * sum[j]) l++;
            
            int k = Q[l];
            f[i][j] = f[i - 1][k] + sqr(sum[j] - sum[k]);
        }
    }
}
 
int main(){
    scanf("%d%d", &n, &m);
 
    sum[0] = 0;
    for(int i = 1; i <= n; i++) scanf("%lld", sum + i), sum[i] += sum[i - 1];
 
    dp();
 
    printf("%lld\n", f[m][n] * m - sqr(sum[n]));
 
    return 0;
}

```
就是这样啦

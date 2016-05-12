---
title: 防御准备 - 斜率优化
toc: true
tags:
  - OI
  - DP
  - 斜率优化
  - BZOJ
permalink: Defence-preparation
date: 2016-04-22 17:33:05
---

#### 【题目描述】

![IMG!](Defence-preparation-img1.jpg)

#### 【题目链接】
[BZOJ 3156](http://www.lydsy.com/JudgeOnline/problem.php?id=3156) 防御准备

<!--more-->

#### 【解题思路】
首先将整个$ a\_i $反转，然后即可向左考虑。

设$ f[i] $表示前$ i $个检查点，第$ i $个检查点必须建塔的最小花费。

通过考虑前一个塔的位置$ j $，中间全部放木偶，易得转移，
$$ f[i]=min\{\ f[j]+\frac {(i-j)(i-j-1)} 2 j \in [0, i) \}+a[i] $$

然后斜率优化，

设决策$ j $的决策费用为$F(j)$，展开，并尽量分离$i, j$，得，

$$ F(j) = h(i) + g(i) - ij $$

其中，
$$ h(i) = \frac {i^2 - i} 2 + a[i] $$
$$ g(j) = \frac {j^2 + j} 2 + f[j]$$

如果有决策$ j\_1 < j\_2 $，那么将$ F(j\_2) < F(j\_1) $的充要条件写作斜率的形式，得

$$ Slope(j\_1, j\_2) = \frac {g(j\_1) - g(j\_2)} {i - j} < i $$

因为$ i $单调增，所以维护$Slope(j\_k, j\_{k + 1}) > i$，最优决策取队首。

并且维护斜率单调增。

#### 【AC代码】

注意最后一个DP值不一定最优，所以需再统计答案。

```c++
#include <cstdio>
#include <algorithm>
#include <climits>
 
const int MAXN = 1000061;
typedef long long int64;
 
int n;
int a[MAXN];
int64 f[MAXN];
 
inline int64 g(int64 i){
    return f[i] + (i * i + i) / 2;
}
 
inline int64 h(int64 i){
    return (i * i - i) / 2 + a[i];
}
 
inline double slope(int64 i, int64 j){
    return (double)(g(i) - g(j)) / (i - j);
}
 
inline void dp(){
    static int Q[MAXN];
    int l = 0, r = -1;
 
    f[0] = a[0], Q[++r] = 0;
 
    for(int i = 1; i < n; i++){
        while(l < r && slope(Q[l], Q[l + 1]) < i) l++;
 
        int j = Q[l];
        f[i] = g(j) + h(i) - (int64)i * j;
 
        while(l < r && slope(Q[r - 1], Q[r]) > slope(Q[r], i)) r--;
        Q[++r] = i;
    }
}
 
inline void updateMin(int64 &x, int64 y){
    x = std::min(x, y);
}
 
int main(){
    scanf("%d", &n);
    for(int i = n - 1; i >= 0; i--) scanf("%d", a + i);
 
    dp();
     
    int64 ans = LLONG\_MAX;
    for(int i = 0; i < n; i++){
        updateMin(ans, f[i] + (int64)(n - i - 1) * (n - i) / 2);
    }
 
    printf("%lld\n", ans);
 
    return 0;
}

```
就是这样啦

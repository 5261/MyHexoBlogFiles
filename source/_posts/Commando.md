---
title: 特别行动队 - 斜率优化
toc: true
date: 2016-04-20 13:43:53
tags:
  - OI
  - DP
  - 斜率优化
  - APIO
  - BZOJ

permalink: Commando
---

#### 【题目描述】
![IMG1](Commando-img-1.jpg)

#### 【题目链接】

[BZOJ 1911](http://www.lydsy.com/JudgeOnline/problem.php?id=1911)特别行动队 【APIO2010】

<!--more-->

#### 【解题思路】
动态规划，设$ f[i] $表示把前$ i $个士兵分成若干特别行动队能获得的最大战力，另以$ s\_i $表示$ x\_i $的前缀和，通过考虑和第$ i $个士兵在一起的士兵，易得状态转移：

$$ f[i] = max\\{\\ f[j - 1] + a(s\_i - s\_{j - 1})^2 + b(s\_i - s\_{j - 1}) + c\ \\} \\ j \in [1, i]$$

设决策$ j $的决策收益为$ F(j) $，即

$$ F(j) = f[j - 1] + a(s\_i - s\_{j - 1})^2 + b(s\_i - s\_{j - 1}) + c $$

展开，令$ i $和$ j $尽量分离，得到，

$$ F(j) = h(i) + g(j) - 2as\_is\_{j - 1} $$

其中，

$$
h(i) = as\_i^2 + bs\_i + c \\
g(j) = f[j - 1] + as\_{j - 1}^2 - bs\_{j - 1}
 $$

然后考虑对于决策$j\_1 < j\_2$，$F(j\_2) > F(j\_1)$的充要条件化为斜率形式为：

$$ Slope(j\_1, j\_2) = \frac {g(j\_2) - g(j\_1)} {s\_{j\_1 - 1} - s\_{j\_2 - 1}} > 2as\_i$$

然后斜率优化，

因为$ 2as\_i $是单调减的，所以要维护任意$ Slope(j\_k, j\_{k + 1}) < 2as\_i$，则最优决策在队首。

另维护斜率单调减，简单分析易得。

#### 【AC代码】

```c++
#include <cstdio>
 
const int MAXN = 1000061;
typedef long long int64;
 
int n;
int64 a, b, c;
int64 w[MAXN + 1];
 
int64 f[MAXN + 1];
 
inline int64 sqr(int64 x){
    return x * x;
}
 
inline int64 g(int i){
    return f[i - 1] + a * sqr(w[i - 1]) - b * w[i - 1];
}
 
inline int64 h(int i){
    return a * sqr(w[i]) + b * w[i] + c;
}
 
inline double slope(int i, int j){
    return (double)(g(j) - g(i)) / (w[j - 1] - w[i - 1]);
}

inline void dp(){
    int Q[MAXN + 1];
    int l = 0, r = -1;
    for(int i = 1; i <= n; i++){
        while(l < r && slope(Q[r - 1], Q[r]) < slope(Q[r], i)) r--;
        Q[++r] = i;
        while(l < r && slope(Q[l], Q[l + 1]) > 2 * a * w[i]) l++;
        
        int j = Q[l];
        f[i] = h(i) + g(j) - 2 * a * w[i] * w[j - 1];
    }
}
 
int main(){
    scanf("%d", &n);
    scanf("%lld%lld%lld", &a, &b, &c);
 
    w[0] = 0;
    for(int i = 1; i <= n; i++) scanf("%lld", w + i), w[i] += w[i - 1];
 
    dp();
 
    printf("%lld\n", f[n]);
 
    return 0;
}

```

#### 【题外话】

题号很巧，让我想起了 柯尔特M1911，在此致敬。

此题A掉了好开心，记得还是去年在北京听课，讲到斜率优化时还是满脸懵逼，几乎没有勇气听下去，只觉得这是极其高深莫测的技能，高不可攀，讲完后进行DP测试，T2的原型就是这题，仅有些许改动（其实也就改了题面，特别行动队改成了喵星人什么的），当时完全不懂，连暴力DP也写不出。

而今日却拿下了此题，却也感觉并不是那么难的样子，实为感慨。

细细算来学OI也有将近半年了吧，我也不知最后会怎样，但现在看来一切还好吧，至少能真真切切的看到自己的进步，自己的成长，尽管差距依旧很大，但总是要给自己些鼓励的不是嘛。

或许难料，只愿尽心。

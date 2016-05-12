---
title: 仓库建设 - 斜率优化
toc: true
tags:
  - OI
  - DP
  - 斜率优化
  - ZJOI
  - BZOJ
date: 2016-04-22 17:24:18
permalink: Storage-ZJOI2007
---

#### 【题目描述】
L公司有N个工厂，由高到底分布在一座山上。如图所示，工厂1在山顶，工厂N在山脚。

由于这座山处于高原内陆地区（干燥少雨），L公司一般把产品直接堆放在露天，以节省费用。

突然有一天，L公司的总裁L先生接到气象部门的电话，被告知三天之后将有一场暴雨，于是L先生决定紧急在某些工厂建立一些仓库以免产品被淋坏。

由于地形的不同，在不同工厂建立仓库的费用可能是不同的。

第i个工厂目前已有成品$ P\_i $件，在第$ i $个工厂位置建立仓库的费用是$ C\_i $。

对于没有建立仓库的工厂，其产品应被运往其他的仓库进行储藏，而由于L公司产品的对外销售处设
置在山脚的工厂N，故产品只能往山下运（即只能运往编号更大的工厂的仓库），

当然运送产品也是需要费用的，假设一件产品运送1个单位距离的费用是1。假设建立的仓库容量都都是足够大的，可以容下所有的产品。

你将得到以下数据：
1：工厂$ i $距离工厂1的距离$ X\_i $（其中$ X\_1 $=0）;
2：工厂$ i $目前已有成品数量$ P\_i $;
3：在工厂$ i $建立仓库的费用$ C\_i $;

请你帮助L公司寻找一个仓库建设的方案，使得总的费用（建造费用+运输费用）最小。

#### 【题目链接】
[BZOJ 1096](http://www.lydsy.com/JudgeOnline/problem.php?id=1096) 仓库建设 【ZJOI 2007】

<!--more-->

#### 【解题思路】

这题很像一道另一道题目嘛，【CEOI 2004】锯木厂选址。

首先考虑一个暴力DP，设$ f[i] $表示只在前$ i $个工厂，第$ i $个工厂必须建造仓库的最小花费。

通过考虑上一个仓库的建造位置$ j $，可得，

$$ f[i] = min\{\ f[j] + cost(j, i), \ j \in [0, i)\} + c\_i$$

其中$ cost(j, i) $表示将$ [j + 1, i] $这一段存入$ i $的花费。

如何尽可能快地计算这个花费呢？我们借助前缀和的思想。

设$ w\_i $为$P\_i$的前缀和。
如果假定这些产品都是从起点运到$ i $的，那么总花费就是$ (w\_i - w\_j)x\_i $，但因为这些产品并非是从起点运过来的，所以对于每组产品$ t $可节省$x\_tP\_t$的费用，那么设$ d\_i $为$ x\_iP\_i $的前缀和，则$  cost(j, i) = (w\_i - w\_j)x\_i - (d\_i - d\_j)$，所以我们的转移方程就成了：

$$ f[i] =  min\{\ f[j] + (w\_i - w\_j)x\_i - (d\_i - d\_j), \ j \in [0, i)\} + c\_i$$

接下来就要准备斜率优化咯，展开，尽量分离$i, j$，容易得到，

设决策$ j $的决策费用为$ F(j) $，则，
$$ F(j) = h(i) + g(j) - w\_jx\_i $$
其中，
$$ h(i) = w\_ix\_i - d\_i + c\_i$$
$$ g(j) = f[j] + d\_j $$

然后考虑对于决策$j\_1 < j\_2$，$F(j\_2) < F(j\_1)$的充要条件化为斜率形式为：

$$ Slope(j\_1, j\_2) = \frac {g(j\_1) - g(j\_2)} {w\_{j\_1} - w\_{j\_2}} < x\_i$$

因为$ x\_i $递增，所以维护 $ Slope(j\_k, j\_{k + 1}) > x\_i $，最优决策取队首。

同时维护斜率单调增。

#### 【AC代码】
注意边界。
```c++
#include <cstdio>
 
const int MAXN = 1000061;
typedef long long int64;
 
int n;
int x[MAXN + 1], a[MAXN + 1], c[MAXN + 1];
 
int64 w[MAXN + 1], d[MAXN + 1];
 
int64 f[MAXN + 1];
 
inline int64 g(int i){
    return f[i] + d[i];
}
 
inline int64 h(int i){
    return x[i] * w[i] - d[i] + c[i];
}
 
inline double slope(int i, int j){
    return (double)(g(i) - g(j)) / (w[i] - w[j]);
}
 
inline void dp(){
    static int Q[MAXN];
    int l = 0, r = -1;
 
    f[0] = 0, Q[++r] = 0;
 
    for(int i = 1; i <= n; i++){
        while(l < r && slope(Q[l], Q[l + 1]) < x[i]) l++;
 
        int j = Q[l];
        f[i] = h(i) + g(j) - x[i] * w[j];
 
        while(l < r && slope(Q[r - 1], Q[r]) > slope(Q[r], i)) r--;
        Q[++r] = i;
    }
}
 
int main(){
    // freopen("storage.in", "r", stdin), freopen("storage.out", "w", stdout);
 
    scanf("%d", &n);
    for(int i = 1; i <= n; i++) scanf("%d%d%d", x + i, a + i, c + i);
 
    w[0] = d[0] = 0;
    for(int i = 1; i <= n; i++){
        w[i] = w[i - 1] + a[i];
        d[i] = d[i - 1] + (int64)x[i] * a[i];
    }
 
    dp();
 
    printf("%lld\n", f[n]);
 
    // fclose(stdin), fclose(stdout);
    return 0;
}
```
就是这样啦

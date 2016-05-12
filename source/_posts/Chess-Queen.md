---
title: 象棋中的皇后 - 计数
toc: true
tags:
  - OI
  - 数学
  - 计数

permalink: Chess-Queen
date: 2016-04-24 09:25:14
---

#### 【题目描述】
在 n x m 棋盘上放两个皇后，求使她们能相互攻击的方案数。

#### 【题目链接】
[COGS 1447](http://cogs.top/cogs/problem/problem.php?pid=1474) 象棋中的皇后

<!--more-->

#### 【解题思路】

因为只有两只皇后，所以互相攻击的情形只有在同一行、同一列或是同一对角线，而且这些情况间是没有交集的，所以可以分开算，然后应用加法原理。

考虑同一行怎么算，第一只皇后有$ nm $个选择，第二只皇后有$ m - 1 $个选择，根据乘法原理，方案数为$nm(m - 1)$。

同一列的情形和同一行类似，易得方案数为$ nm(n - 1) $。

接下来我们来考虑副对角线（即方向为左下到右上的对角线），不妨设$ n <= m $，看图：

![IMG1](Chess-Queen-img1.png)

在此图中，长度相等的对角线用了同一种颜色标示。

可以看到，自左至右对角线的长度为：
$ 1,2,3 \cdots n - 1,\underbrace { n, n, n, \cdots ,n}_{m - n + 1}, n - 1,\cdots , 3, 2, 1 $

若单独考虑一条长度为$ x $的对角线，方案数显然为$ x(x - 1) $，根据加法原理，把所有对角线的方案数加起来即可，考虑到还有另一条对角线，所以答案再整个乘以2。

$$ ans = 2\left( 2\sum_{i = 1}^{n - 1}i(i - 1) + (m - n + 1)n(n - 1) \right)$$

其中，

$$
\begin{align\*}
 \sum\_{i = 1}^{n - 1}i(i - 1) 
 & = \sum\_{i = 1}^{n - 1}i^2 - \sum\_{i = 1}^{n - 1}i \\\\
& = \frac {n(n - 1)(2n - 1)} 6 - \frac {n(n - 1)} 2 \\\\
& = \frac {n(n - 1)(2n - 4)} 3
\end{align\*}
$$
（以上依据幂和公式）
故，
$$ 
\begin{align\*}
ans 
& = 2\left( \frac {n(n - 1)(2n - 4)} 3 + (m - n + 1)n(n - 1) \right) \\\\
& = \frac {2n(n - 1)(3m - n - 1)} 3
\end{align\*}
$$

然后三种情况全部加起来就好咯。

#### 【AC代码】
```c++
#include <cstdio>
#include <algorithm>
 
typedef unsigned long long uint64;
 
int main(){
    // freopen("chessqueen.in", "r", stdin), freopen("chessqueen.out", "w", stdout);
 
    uint64 n, m;
 
    scanf("%llu%llu", &n, &m);
 
    if(n > m) std::swap(n, m);
    printf("%llu\n", n * m * (m + n - 2) + 2 * n * (n - 1) * (3 * m - n - 1) / 3);
 
    // fclose(stdin), fclose(stdout);
    return 0;
}
```
就是这样啦。

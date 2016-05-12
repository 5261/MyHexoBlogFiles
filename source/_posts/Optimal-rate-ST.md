title: 最优比率生成树
toc: true
date: 2016-03-10 22:00:55
tags:
  - OI
  - 图论
  - 生成树
  - 0-1分数规划
  - 二分
premalink: Optimal-rate-ST
---

### 【问题描述】
给定一个图$G = （V,E）$，每一条边$e$均有两个属性收益$benifit$和花费$cost\,$并记为$e\.benifit$和$e\.cost$;
对于该图的一颗生成树T，我们定义其比率为：
$$ f\(T\) = \\frac \{\\sum \_\{e \\in T\} \\ e\.benifit\} \{\\sum \_\{e \\in T\} \\ e\.cost\} $$
现在我们的任务是在图G上求一颗生成树$T$，使得$f\(T\)$最小（当然也可以求最大）。

<!--more-->

### 【解题思路】

任务：
$$Minimize \~\~ f\(T\) = k$$

最优性问题，考虑转化为判定性问题，对于比率 $k$我们设置一个询问$ask（k）$，意为:是否存在一颗生成树$T$， 使得$f\(T\) \\ge k$ ?如果我们能够解决这个询问，我们就把最优性问题转化为判定性问题了,那么求$f\(T\)$最值也就没问题了，就可以选择二分。

如何解决这个询问呢，来看一下$f\(T\) \\ge k$的等价变形:
 $$ \\begin\{align\*\} 
 & f\(T\) = \\frac \{\\sum \_\{e \\in T\} \\ e\.benifit\} \{\\sum \_\{e \\in T\} \\ e\.cost\} \\ge k 
 \\\\ & \\Leftrightarrow \\ \{\\sum \_\{e \\in T\} \\ e\.benifit\} \\ge k \* \{\\sum \_\{e \\in T\} \\ e\.cost\}
 \\\\ & \\Leftrightarrow \\ \{\\sum \_\{e \\in T\} \\ e\.benifit\} \- k \* \{\\sum \_\{e \\in T\} \\ e\.cost\} \\ge 0
 \\\\ & \\Leftrightarrow \\ \{\\sum \_\{e \\in T\} \\ e\.benifit\} \- \{\\sum \_\{e \\in T\} \\ k \* e\.cost \\ge 0\}
 \\\\ & \\Leftrightarrow \\ \{\\sum \_\{e \\in T\} \\ e\.benifit \- k \* e\.cost\} \\ge 0
 \\end\{align\*\}$$
 我令
 $$h\(k\) = min\\\{ \\ \{\\sum \_\{e \\in T\} \\ e\.benifit \- k \* e\.cost\}\\ \\\}$$
 那么我们的任务，就是判定:
 $$h\(k\) \\ge 0$$
 看出来了吗，$h\(k\)$实际上就是图中每条边以 $e\.benifit \- k \* e\.cost$ 为边权的最小生成树的权和，这样我们就可以回答一开始的询问了。显然h(k)这个函数是单调的，所以我们可以使用二分来查找k的值，问题就得到了解决。
 
 ---
 
### 【分数规划】
 以上便是一个0-1分数规划的经典例子，展示了分数规划的通常思路与过程，对于分数规划，一般化的，我们有:
 >分数规划的一般形式:
$$Minimize \\ k = f\(x\) = \\frac \{a\(x\)\} \{b\(x\)\} \~\~ \(x \\in S\)$$
分数规划的构造的解决函数:
$$h\(k\) = min\\\{\\ a\(x\) \- k \* b\(x\)\\ \\\}$$
解决函数性质（不证）:
1. $h\(k\)$是一个严格单调递减函数
2. 设$k^\*$为该规划的最优解，则有:
$$ \\left\\\{
\\begin\{aligned\}
g\(k\) = 0 \\Leftrightarrow k = k^\* \\\\
g\(k\) <  0 \\Leftrightarrow k \> k^\* \\\\
g\(k\) \> 0 \\Leftrightarrow k <  k^\*
\\end\{aligned\}
\\right\.
$$
—— 以上摘自 胡伯涛Amber论文《最小割模型在信息学竞赛中的应用》

---
解决分数规划，除了二分，还有一个很重要的方法称Dinkelbach，效率上也比二分要高，但限于水平，本文不再介绍。

---
### 【例题】
来道最优比率生成树例题：
[POJ 2728](http://poj.org/problem?id=2728) Desert King
很裸的最优比率生成树，以两点间欧几里得距离为花费$cost$，高度差为收益$benifit$，求解出最优比率即为答案。
另外，此图是完全图，有$m = O\(n^2\)$，所以最好采用$O\(n^2\)$的朴素Prim算法求解最小生成树，其他像优化的Prim和Kruskal的复杂度都难以承受。
【AC代码】
```c++
#include <cstdio>
#include <cfloat>
#include <cmath>
#include <cstring>

#define MAXV 1526
#define EPS 1e-5

int n;
double Benifit[MAXV][MAXV], Cost[MAXV][MAXV];
double G[MAXV][MAXV];
int x[MAXV], y[MAXV], h[MAXV];

bool used[MAXV];
double low[MAXV];

double sum;

inline double prim(int s = 0){
    int cur;
    double ans = 0;

    memset(used, false, sizeof used);
    cur = s, used[s] = true;
    for(int i = 0; i < n; i++) if(i != s) low[i] = G[cur][i];

    for(int t = 0; t < n - 1; t++){
        double min = DBL_MAX;
        for(int i = 0; i < n; i++){
            if(!used[i] && low[i] < min){
                min = low[i], cur = i;
            }
        }
        ans += min;
        used[cur] = true;
        for(int i = 0; i < n; i++){
            if(!used[i] && G[cur][i] < low[i]){
                low[i] = G[cur][i];
            }
        }
    }

    return ans;
}

inline double dist(int i, int j){
    return sqrt((x[i] - x[j]) * (x[i] - x[j]) + (y[i] - y[j]) * (y[i] - y[j]));
}

inline double divide(){
    double l = 0, r = 10000000;
    while(r - l > EPS){
        double mid = (l + r) / 2;
        for(int i = 0; i < n; i++) for(int j = 0; j < n; j++){
            G[i][j] = G[j][i] = Benifit[i][j] - mid * Cost[i][j];
        }
        double h = prim();
        if(h > EPS) l = mid;
        else r = mid;
    }
    return (l + r) / 2;
}

int main(){
    while(5261){
        scanf("%d", &n);
        if(n == 0) break;
        for(int i = 0; i < n; i++) scanf("%d%d%d", x + i, y + i, h + i);
        for(int i = 0; i < n; i++) for(int j = 0; j < n; j++){
            Cost[i][j] = dist(i, j);
            Benifit[i][j] = fabs(h[i] - h[j]);
        }
        printf("%.3f\n", divide());
    }
    return 0;
}
```
就是这样啦
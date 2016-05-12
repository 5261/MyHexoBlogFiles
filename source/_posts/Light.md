---
title: 小凸玩密室 - 树形DP
toc: true
date: 2016-03-25 20:53:34
tags:
  - OI
  - DP
  - 树形DP
  - 完全二叉树
  - SCOI
  - 安师大附中集训
permalink: Light
---

### 【问题描述】

密室是一棵有 $n$ 个节点的完全二叉树，每个节点有一个灯泡。点亮所有灯泡即可逃出密室。
每个灯泡有个权值 $A\_i$，每条边也有个权值 $B\_i$。
点亮第 $1$ 个灯泡不需要花费，之后每点亮一 个新的灯泡V的花费，等于上一个被点亮的灯泡 $U$ 到这个点 $V$ 的距离 $D\_\{u,v\}$，乘以这个点的权值 $A\_v$。
在点灯的过程中，要保证任意时刻所有被点亮的灯泡必须连通，在点亮一个灯泡后必须先点亮其子树所有灯泡才能点亮其他灯泡。
请告诉他们，逃出密室的最少花费是多少。

<!--more-->

### 【题目链接】
[BZOJ 4446](http://www.lydsy.com/JudgeOnline/problem.php?id=4446) 小凸玩密室 SCOI2015

### 【解题思路】

自己做的时候并没有什么思路QwQ。
之后看了神奇的题解后想了好久似乎明白些了。

动态规划。

令 $f\_\{v,i\}$ 表示从$v$开始点亮，$v$的子树还未点亮，$v$的子树全部点亮之后要走到深度为$i$的祖先的另外一个孩子（记作$t$）的最小代价。

转移：

当$v$为叶子时，直接走过去。

$$ f\_\{v, i\} = A\_t \* dist(v, t)$$

当$v$只有左孩子时，先走到左孩子，再从左孩子走过去。

$$ f\_\{v, i\} = A\_\{lc\} \* B\_\{lc\} + f\_\{lc, i\}$$

当$v$有两个孩子时，两种走法，先走左孩子，然后从左孩子走到右孩子，再从右孩子走过去，先走右孩子亦然，取较优的。

$$
f\_\{v, i\} = min:
    \\\\
    \\\\ A\_\{lc\} \* B\_\{lc\} + f\_\{lc,\\ depth(v)\} + f\_\{rc, i\},
    \\\\ A\_\{rc\} \* B\_\{rc\} + f\_\{rc,\\ depth(v)\} + f\_\{lc, i\}.
$$

至此，$f\_\{v, i\}$的问题已全部转化为子问题，可以由孩子递推而来。

然而还没完，这只是另一个DP的基础。

另一个DP：

设 $g\_\{v, i\}$ 表示走完v的子树再走到v的深度为i的祖先（记作$t$）的最小代价，转移和$f$基本同理：

叶子:

$$
g\_\{v,i\} = A\_\{t\} \* dist(v, t)
$$
只有左孩子:

$$
g\_\{v, i\} = A\_\{lc\} \* B\_\{lc\} + g\_\{lc, i\}
$$

左右孩子都有:

$$
g\_\{v, i\} = min:
    \\\\
    \\\\ A\_\{lc\} \* B\_\{lc\} + f\_\{lc, depth(v)\} + g\_\{rc, i\},
    \\\\ A\_\{rc\} \* B\_\{rc\} + f\_\{rc, depth(v)\} + g\_\{lc, i\}.
$$

特殊的，当状态中$i = 0$时表示停在任意位置，并没有深度为0的点，因此我们计算距离时设为零。这也是不违背常理的。

于是DP完惹，考虑如何算答案，如果从根开始，答案即为 $g\_\{1, 0\}$，除此之外，枚举所有非根的节点$v$作为起点，最优方案为先走v子树，再走v的兄弟子树，再走v的父亲的兄弟子树……，直到走到根，因为树高是$O(logn)$的，所以就这样模拟走着算就好。

取最小值，over。

### 【Tricks】

一些关于完全二叉树的计算， 对于点 $v$ :

 - 左孩子为 $v <\\!\\!< 1$.
 - 右孩子为 $v <\\!\\!< 1\ |\ 1$
 - 父亲为 $v >\\!\\!> 1$   
 - 向上$k$层的祖先为 $v >\\!\\!> k$
 - 兄弟为 $v\\ xor\\ 1$
 - $ 1 \\leq v \\leq n$，否则$v$不存在

### 【AC代码】
```c++
#include <cstdio>
#include <algorithm>
 
#define MAXN 200000 + 1
#define MAXLOGN 18 + 1
 
#define int64 long long
 
int n, depth[MAXN];
int64 a[MAXN], b[MAXN], d[MAXN];
int64 f[MAXN][MAXLOGN], g[MAXN][MAXLOGN];
 
inline void dp(){
    for(int v = n; v; v--){
        for(int i = depth[v] - 1; i >= 0; i--){
            int lc = v << 1, rc = lc | 1;
            int target = v >> (depth[v] - i - 1) ^ 1;
            if(lc > n){
                f[v][i] = a[target] * (d[v] + d[target] - (d[target >> 1] << 1));
            }
            else if(rc > n){
                f[v][i] = f[lc][i] + a[lc] * b[lc];
            }
            else{
                f[v][i] = std::min(
                    a[lc] * b[lc] + f[lc][ depth[v] ] + f[rc][i],
                    a[rc] * b[rc] + f[rc][ depth[v] ] + f[lc][i]
                );
            }
        }
    }
    for(int v = n; v; v--){
        for(int i = depth[v]; i >= 0; i--){
            int lc = v << 1, rc = lc | 1;
            int target = v >> (depth[v] - i);
            if(lc > n){
                g[v][i] = a[target] * (d[v] - d[target]);
            }
            else if(rc > n){
                g[v][i] = g[lc][i] + a[lc] * b[lc];
            }
            else{
                g[v][i] = std::min(
                    a[lc] * b[lc] + f[lc][ depth[v] ] + g[rc][i],
                    a[rc] * b[rc] + f[rc][ depth[v] ] + g[lc][i]
                );
            }
        }
    }
}
 
inline int64 calc(int v){
    int64 ans = g[v][ depth[v] - 1];
    for(; v != 1; v >>= 1){
        int brother = v ^ 1, father = v >> 1;
        if(brother > n){
            ans += a[father >> 1] * b[father];
        }
        else{
            ans += a[brother] * b[brother] + g[brother][ depth[father] - 1 ];
        }
    }
    return ans;
}
 
inline int64 solve(){
    dp();
    int64 ans = g[1][0];
    for(int i = 2; i <= n; i++) ans = std::min(ans, calc(i));
    return ans;
}
 
int main(){
    scanf("%d", &n);
    for(int i = 1; i <= n; i++) scanf("%lld", a + i);
    for(int i = 2; i <= n; i++) scanf("%lld", b + i);
 
    depth[1] = 1, d[1] = 0;
    for(int i = 2; i <= n; i++){
        depth[i] = depth[i >> 1] + 1;
        d[i] = d[i >> 1] + b[i];
    }
 
    printf("%lld\n", solve());
 
    return 0;
}
```
就这样啦

---
title: 选课 - 树形DP
toc: true
tags:
  - OI
  - DP
  - 树形DP
  - CTSC
date: 2016-04-06 21:49:32
permalink: Selective-course
---

### 【题目描述】

学校实行学分制。每门的必修课都有固定的学分，同时还必须获得相应的选修课程学分。学校开设了N（N<300）门的选修课程，每个学生可选课程的数量M是给定的。学生选修了这M门课并考核通过就能获得相应的学分。 

在选修课程中，有些课程可以直接选修，有些课程需要一定的基础知识，必须在选了其它的一些课程的基础上才能选修。每门课的直接先修课最多只有一门。两门课也可能存在相同的先修课。每门课都有一个课号，依次为1，2，3...。

你的任务是为自己确定一个选课方案，使得你能得到的学分最多，并且必须满足先修课优先的原则。假定课程之间不存在时间上的冲突。

### 【题目链接】
[CodeVS 1378](http://codevs.cn/problem/1378/) 选课 【CTSC 1997】

<!--more-->

### 【解题思路】

课程之间的依赖关系形成了森林，我们添加一个虚拟节点（本题中以0表示），将其作为所有无依赖的课程的依赖课程。

这样就变成了树上的问题，考虑使用树形DP来解决。

设$f[v][m]$表示，在$v$的子树和$v$的兄弟子树中，选修$m$门课程获得的最大学分。

对于一门课程，我们有两种决策。

一种是放弃该课程，将资源$m$全部传递给下一个兄弟。

另一种是选择该课程，并在其子树中分配一定资源$k$，剩余的资源$m - k - 1$传递给下一个兄弟。

转移方程为：

$$
f[v][m] = max:
\\\\ max\\{\\ f[ v.child ][k] + f[ v.next ][m - k - 1]\\ \\}\\ k \\in [0, m) + v.w
\\\\ f[ v.next ][m]
$$

### 【AC代码】

为方便，采取记忆化搜索实现。
注意最后的答案为$f[0][m + 1]$，这是因多了一个虚拟节点。

```c++
#include <cstdio>
#include <cstring>
#include <algorithm>

#define MAXN 1000
#define MAXM 1000

struct Node{
    int w;
    Node *child, *next;

    int f[MAXM + 2];
    bool solved[MAXN + 2];

    Node() : w(0), child(NULL), next(NULL) {
        memset(solved, 0, sizeof solved);
    }

    int solve(int m){
        if(!this || m < 0) return 0;

        if(!solved[m]) return f[m];
        else solved[m] = true;

        f[m] = 0;
        for(int k = 0; k < m; k++){
            f[m] = std::max(f[m], child->solve(k) + next->solve(m - k - 1) + w);
        }
        f[m] = std::max(f[m], child->solve(m));

        return f[m];
    }

} vs[MAXN + 1];

inline void addChild(int a, int b){
    Node *u = vs + a, *v = vs + b;
    v->next = u->child, u->child = v;
}

int main(){
    int n, m;

    scanf("%d%d", &n, &m);
    for(int i = 1; i <= n; i++){
        int v;
        scanf("%d%d", &vs[i].w, &v);
        addChild(v, i);
    }

    printf("%d\n", vs->solve(m + 1));
    return 0;
}

```
就是这样啦

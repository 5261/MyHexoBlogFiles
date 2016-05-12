---
title: 物流运输 - DP + 最短路
toc: true
tags:
  - OI
  - DP
  - 最短路
  - BZOJ
date: 2016-04-28 09:26:15
permalink: Trans-ZJOI2006
---

最近懒~~（颓废）~~了好多，又不太更新Blog了......

#### 【题目描述】
物流公司要把一批货物从码头A运到码头B。由于货物量比较大，需要n天才能运完。

货物运输过程中一般要转停好几个码头。物流公司通常会设计一条固定的运输路线，以便对整个运输过程实施严格的管理和跟踪。

由于各种因素的存在，有的时候某个码头会无法装卸货物。这时候就必须修改运输路线，让货物能够按时到达目的地。

但是修改路线是一件十分麻烦的事情，会带来额外的成本。

因此物流公司希望能够订一个n天的运输计划，使得总成本尽可能地小。

#### 【题目链接】
[BZOJ 1003](http://www.lydsy.com/JudgeOnline/problem.php?id=1003) 物流运输 【ZJOI 2016】

<!--more-->

#### 【解题思路】

动态规划，设$ f[i] $表示前$ i $天的最小成本，转移如下：

如果这$ i $天不改变运输计划，那么：
$$ f[i] = cost(1, i) $$

如果改变，那么枚举上一次改变是在第$ j $天，那么：
$$ f[i] = min\{\ f[j] + cost(j + 1, i) + k, j \in [2, i) \} $$

两种情况取个$ min $咯。

其中$ k $是改变运输计划的代价（输入中给出），$ cost(x, y) $表示从第$ x $天到第$ y $天不改变运输计划的最小花费，求一下最短路就可以知道啦。

#### 【AC代码】
```c++
#include <cstdio>
#include <algorithm>
#include <climits>
#include <queue>
 
#define MAXN 100
#define MAXM 20
 
struct Range{
    int l, r;
};
 
struct Node{
    struct Edge *edges;
 
    int dist;
    bool inQue;
 
    bool invaild;
 
    std::vector<Range> ranges;
 
    Node() : edges(NULL) {}
 
} nodes[MAXM];
 
struct Edge{
    Node *to;
    Edge *next;
    int w;
 
    Edge(Node *from, Node *to, int w) : to(to), next(from->edges), w(w) {}
};
 
inline void addUEdge(int a, int b, int w){
    Node *u = nodes + a, *v = nodes + b;
    u->edges = new Edge(u, v, w), v->edges = new Edge(v, u, w);
}
 
int m, k;
Node *s, *t;
 
inline int cost(int l, int r){
    for(Node *v = nodes; v != nodes + m; v++){
        v->dist = INT_MAX, v->inQue = false;
        v->invaild = false;
        for(std::vector<Range>::iterator range = v->ranges.begin(); range != v->ranges.end(); range++){
            if(std::max(l, range->l) <= std::min(r, range->r)){
                v->invaild = true;
                break;
            }
        }
    }
 
    std::queue<Node*> Q;
 
    Q.push(s);
    s->dist = 0, s->inQue = true;
 
    while(!Q.empty()){
        Node *v = Q.front(); Q.pop();
        v->inQue = false;
 
        for(Edge *e = v->edges; e; e = e->next) if(!e->to->invaild){
            if(e->to->dist > v->dist + e->w){
                e->to->dist = v->dist + e->w;
                if(!e->to->inQue){
                    Q.push(e->to);
                    e->to->inQue = true;
                }
            }
        }
    }
 
    if(t->dist == INT_MAX) return INT_MAX;
    else return t->dist * (r - l + 1);
}
 
inline void updateMin(int &x, int y){
    x = std::min(x, y);
}
 
int n;
int f[MAXN + 1];
 
inline int dp(){
    for(int i = 1; i <= n; i++){
        f[i] = cost(1, i);
        for(int j = 2; j < i; j++){
            int c = cost(j + 1, i);
            if(c == INT_MAX) continue;
            else updateMin(f[i], f[j] + c + k);
        }
    }
    return f[n];
}
 
int main(){
    // freopen("bzoj_1003.in", "r", stdin), freopen("bzoj_1003.out", "w", stdout);
 
    scanf("%d%d%d", &n, &m, &k);
    s = nodes, t = nodes + m - 1;
 
    int e;
    scanf("%d", &e);
    for(int i = 0; i < e; i++){
        int u, v, w;
        scanf("%d%d%d", &u, &v, &w);
        u--, v--;
        addUEdge(u, v, w);
    }
 
    int d;
    scanf("%d", &d);
    for(int i = 0; i < d; i++){
        int x, l, r;
        scanf("%d%d%d", &x, &l, &r);
        x--;
        (nodes + x)->ranges.push_back((Range){l, r});
    }
 
    printf("%d\n", dp());
 
    // fclose(stdin), fclose(stdout);
    return 0;
}
```

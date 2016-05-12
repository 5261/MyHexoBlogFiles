---
title: 狼抓兔子 - 最小割 / 平面图 最短路
toc: true
date: 2016-04-13 10:09:17
tags: 
  - OI
  - 网络流
  - 最小割
  - 平面图
  - 最短路
  - BZOJ

permalink: Wolf-rabbit
---

#### 【题目描述】
现在小朋友们最喜欢的"喜羊羊与灰太狼",话说灰太狼抓羊不到，但抓兔子还是比较在行的，而且现在的兔子还比较笨，它们只有两个窝，现在你做为狼王，面对一个网格的地形。

左上角点为(1,1),右下角点为(N,M)，有以下三种类型的道路：
(x, y)<=>(x + 1, y) (x, y)<=>(x, y + 1) (x,y)<=>(x + 1, y + 1)

道路上的权值表示这条路上最多能够通过的兔子数，道路是无向的。

左上角和右下角为兔子的两个窝，开始时所有的兔子都聚集在左上角(1,1)的窝里，现在它们要跑到右下解(N,M)的窝中去，狼王开始伏击这些兔子。

当然为了保险起见，如果一条道路上最多通过的兔子数为K，狼王需要安排同样数量的K只狼，才能完全封锁这条道路。

你需要帮助狼王安排一个伏击方案，使得在将兔子一网打尽的前提下，参与的狼的数量要最小。因为狼还要去找喜羊羊麻烦。

#### 【题目链接】
[BZOJ 1001](http://www.lydsy.com/JudgeOnline/problem.php?id=1001) 狼抓兔子 【BJ 2006】

<!--more-->

#### 【解题思路】

裸的最小割，可以直接上`Dinic`。

现着重说下另一种方法。

该图比较特殊，首先这是一个平面图，而且起点和终点都在平面图的无界面上，我们称这样的平面图为$s-t$ 平面图。

根据平面图及其对偶图的相关知识，$s-t$ 平面图 的最小割问题可以转化为最短路问题。

转化方法，$s$ 到 $t$ 之间连一条线，将原来的无界面分为 $s'$ 和 $t'$两部分，将面看成新图中的结点，原图中的一条边必然同时属于两个面，那么在这两个面代表的结点间连边，边权即为原来的边权，这样，一个原图中 $s$ 到 $t$ 的一个割恰好对应了新图中 $s'$ 到 $t'$的一条路，最小割即对应最短路。

详请参见 国家集训队论文[《两极相通 —— 浅析最大最小定理在信息学竞赛中的应用》](http://wenku.baidu.com/link?url=t57YBwXIkh7Fq0z5Is97rqEwPDzAc8zu2JKK0PVVj3NZEn9shkg9HSyyMbHmzWKptDPfGuqkS8J3ahpj7D4cslnZzj9-S6sZMxqYCTsvhnG)作者 周东
感觉讲的很好，尤其是图片非常形象。

代码：
#### 【Dinic】
感觉到当前弧优化的效果了...
一开始手抖少打了个 `&`，然后`16173 ms`，就T掉了。
然后加上`&`，`2448 ms`，就A掉了。

```c++
#include <cstdio>
#include <climits>
#include <algorithm>
#include <queue>
 
#define MAXN 1000
#define MAXM 1000
#define MAXV MAXN * MAXM
 
struct Node;
struct Edge;
 
struct Node{
    int level;
    Edge *edges, *curEdge;
 
    Node() : edges(NULL) {}
 
} nodes[MAXV];
 
struct Edge{
    Node *to;
    Edge *next, *rev;
    int res;
 
    Edge(Node *from, Node *to, int cap) : to(to), next(from->edges), res(cap) {}
 
};
 
inline void addUEdge(int a, int b, int cap){
    Node *u = nodes + a, *v = nodes + b;
    u->edges = new Edge(u, v, cap);
    v->edges = new Edge(v, u, cap);
    u->edges->rev = v->edges, v->edges->rev = u->edges;
}
 
struct Dinic{
    Node *s, *t;
    int n;
 
    inline bool BFS(){
        for(Node *v = nodes; v != nodes + n; v++) v->level = 0;
 
        std::queue<Node*> Q;
        Q.push(s);
        s->level = 1;
        while(!Q.empty()){
            Node *v = Q.front(); Q.pop();
            for(Edge *e = v->edges; e; e = e->next){
                if(!e->to->level && e->res > 0){
                    e->to->level = v->level + 1;
 
                    if(e->to == t) return true;
                    else Q.push(e->to);
                }
            }
        }
 
        return false;
    }
 
    int DFS(Node *v, int minFlow = INT_MAX){
        if(v == t || !minFlow) return minFlow;
        int total = 0, flow;
 
        for(Edge *&e = v->curEdge; e; e = e->next){
            if(v->level + 1 == e->to->level &&
                (flow = DFS(e->to, std::min(minFlow, e->res))) > 0){
                e->res -= flow, e->rev->res += flow;
                total += flow, minFlow -= flow;
 
                if(!minFlow) break;
            }
        }
 
        return total;
    }
 
    int operator()(int a, int b, int n){
        s = nodes + a, t = nodes + b, this->n = n;
        int ans = 0;
        while(BFS()){
            for(Node *v = nodes; v != nodes + n; v++) v->curEdge = v->edges;
            ans += DFS(s);
        }
        return ans;
    }
 
} dinic;
 
int n, m;
 
inline int id(int i, int j){
    return i * m + j;
}
 
int main(){
    scanf("%d%d", &n, &m);
 
    int w;
 
    for(int i = 0; i < n; i++){
        for(int j = 0; j < m - 1; j++){
            scanf("%d", &w);
            addUEdge(id(i, j), id(i, j + 1), w);
        }
    }
 
    for(int i = 0; i < n - 1; i++){
        for(int j = 0; j < m; j++){
            scanf("%d", &w);
            addUEdge(id(i, j), id(i + 1, j), w);
        }
    }
 
    for(int i = 0; i < n - 1; i++){
        for(int j = 0; j < m - 1; j++){
            scanf("%d", &w);
            addUEdge(id(i, j), id(i + 1, j + 1), w);
        }
    }
 
    int s = id(0, 0), t = id(n - 1, m - 1);
    printf("%d\n", dinic(s, t, n * m));
 
    return 0;
}
```

#### 【最短路】
`Dijkstra`和没加优化的`SPFA`都没跑过`Dinic`，是我写的太挫么...

加了`SLF`的`SPFA`跑了`1580 ms`。

注意想好怎么给面编号，不然加边时会晕...

还有 $m = 1$ 或 $n = 1$ 的情况要特判掉...
```c++
#include <cstdio>
#include <climits>
#include <queue>
#include <algorithm>

#define MAXN 1000
#define MAXM 1000
#define MAXV ((MAXN - 1) * (MAXM - 1) << 1) + 2

struct Node;
struct Edge;

struct Node{
    Edge *edges;
    int dist;
    bool inQue;

    Node() : edges(NULL), dist(INT_MAX), inQue(false) {}

} nodes[MAXV];

struct Edge{
    Node *to;
    Edge *next;
    int w;

    Edge(Node *from, Node *to, int w) : to(to), next(from->edges), w(w) {}
};

inline void addUEdge(int a, int b, int w){
    Node *u = nodes + a, *v = nodes + b;
    u->edges = new Edge(u, v, w), v->edges = new Edge(v, u, w);
    // printf("(%d, %d) -> %d\n", a, b, w);
}

inline int SPFA(int a, int b){
    Node *s = nodes + a, *t = nodes + b;

    std::deque<Node*> Q;
    Q.push_back(s);
    s->dist = 0, s->inQue = true;

    while(!Q.empty()){
        Node *v = Q.front(); Q.pop_front();
        v->inQue = true;
        for(Edge *e = v->edges; e; e = e->next){
            if(e->to->dist > v->dist + e->w){
                e->to->dist = v->dist + e->w;
                if(!e->to->inQue){
                    if(!Q.empty() && e->to->dist < Q.front()->dist) Q.push_front(e->to);
                    else Q.push_back(e->to);
                    e->to->inQue = true;
                }
            }
        }
    }

    return t->dist;
}

int n, m;

inline void updateMin(int &x, int y){
    x = std::min(x, y);
}

inline int id(int i, int j){
    return i * (m - 1 << 1) + (j << 1);
}

inline int solve(){
    int w;

    if(n == 1 && m == 1) return 0;
    else if(n == 1){
        int ans = INT_MAX;
        for(int i = 0; i < m - 1; i++){
            scanf("%d", &w);
            updateMin(ans, w);
        }
        return ans;
    }
    else if(m == 1){
        int ans = INT_MAX;
        for(int i = 0; i < n - 1; i++){
            scanf("%d", &w);
            updateMin(ans, w);
        }
        return ans;
    }

    int s = (n - 1) * (m - 1) << 1, t = s + 1;

    for(int j = 0; j < m - 1; j++){
        scanf("%d", &w);
        addUEdge(s, id(0, j) ^ 1, w);
    }

    for(int i = 1; i < n - 1; i++){
        for(int j = 0; j < m - 1; j++){
            scanf("%d", &w);
            int a = id(i - 1, j), b = id(i, j);
            addUEdge(a, b ^ 1, w);
        }
    }

    for(int j = 0; j < m - 1; j++){
        scanf("%d", &w);
        addUEdge(id(n - 2, j), t, w);
    }

    for(int i = 0; i < n - 1; i++){
        scanf("%d", &w);
        addUEdge(id(i, 0), t, w);

        for(int j = 1; j < m - 1; j++){
            int a = id(i, j), b = id(i, j - 1);
            scanf("%d", &w);
            addUEdge(a, b ^ 1, w);
        }

        scanf("%d", &w);
        addUEdge(s, id(i, m - 2) ^ 1, w);
    }

    for(int i = 0; i < n - 1; i++){
        for(int j = 0; j < m - 1; j++){
            scanf("%d", &w);
            int v = id(i, j);
            addUEdge(v, v ^ 1, w);
        }
    }

    return SPFA(s, t);
}

int main(){
    // freopen("bjrabbit.in", "r", stdin), freopen("bjrabbit.out", "w", stdout);

    scanf("%d%d", &n, &m);

    printf("%d\n", solve());

    // fclose(stdin), fclose(stdout);
    return 0;
}

```
就是这样啦

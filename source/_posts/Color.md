---
title: 染色 - 树链剖分
toc: true
date: 2016-03-25 17:01:45
tags:
  - OI 
  - 数据结构
  - 树链剖分
  - 线段树
  - 安师大附中集训
permalink: Color
---

### 【题目描述】
给定一颗$n$个节点的树，树的节点标号从0开始。每个节点可以是黑色或白色。要求支持以下操作：

- 将x节点涂黑
- 查询节点x到所有黑点的距离之和

<!--more-->

### 【解题思路】

首先想到，在每次询问时，把查询点到所有黑点的距离都全部计算一遍，这显然在时间上是行不通的。

想想我们每次计算的内容，有哪些地方是共通的而不是必须每次重新计算得到的。
设黑点集合$S$，我们要计算的是
$$
ans(x) = \sum_{v \in S} dist(x, v)
$$
我们知道
$$
dist(x, v) = preDist(x) + preDist(v) - 2 * preDist(LCA(x, v))
$$
其中$preDist(x)$表示x到根节点距离。

观察上式，在我们的累加求和过程中，$preDist(x)$ 和$preDist(v)$两项的和都是可以预处理出来的，我们真正需要实时计算的，只是$preDist(LCA(x, v))$一项的和。

树链剖分，每次在将一个点染黑时，将该点到根的路径上所有边权的系数加一（初始系数为零），用线段树维护，在查询时，查询该点到根路径上的带系数边权和，即为所有 $LCA$ 的距离之和。

每次修改都影响 黑点到根的一整条路径上的信息，而询问时是查询 到根的整条路径上的信息，也就是说，每次修改对每次询问造成的实质影响是两条路径的公共部分，即 $LCA$ 到根的的路径，也就是我们需要的信息。

系数含义的实质就是该子树中的黑点个数。

### 【代码】

记得开long long。
树剖写了非递归，感觉也挺自然的。
线段树写的还不是很熟，要多加练习。

```c++
#include <cstdio>
#include <algorithm>
#include <queue>
#include <stack>
#include <iostream>

#define MAXN 1000020

#define int64 long long

struct Node;
struct Path;

struct Node{
    Node *father;
    Node *children, *next;

    bool asked;
    int depth, size;
    Node *maxChild;
    int maxDepth, pos;

    Path *path;

    int w;
    bool blacked;
    int64 dist;

    Node(){
        father = children = next = maxChild = NULL;
        path = NULL;
        asked = blacked = false;
        depth = size = maxDepth = 0;
    }

} vs[MAXN];

int64 n;
Node *root = vs;

inline void addChild(int64 u, int64 v){
    (vs + v)->next = (vs + u)->children;
    (vs + v)->father = vs + u;
    (vs + u)->children = vs + v;
}

#define mid (this->l + this->r >> 1)

struct SegmentTree{
    SegmentTree *lchild, *rchild;
    int l, r;
    int base; // base为带系数和的基
    int lazy; // 区间延迟修改量
    int64 sum; // 该区间带系数和

    void update(){
        sum = base = 0;
        if(lchild) sum += lchild->sum, base += lchild->base;
        if(rchild) sum += rchild->sum, base += rchild->base;
    }

    void pushDown(){
        if(lazy){
            if(lchild) lchild->lazy += lazy, lchild->sum += lazy * lchild->base;
            if(rchild) rchild->lazy += lazy, rchild->sum += lazy * rchild->base;
            lazy = 0;
        }
    }

    SegmentTree(int64 l, int64 r) : l(l), r(r), base(0), lazy(0), sum(0){
        if(r - l == 1) lchild = rchild = NULL;
        else{
            lchild = new SegmentTree(l, mid);
            rchild = new SegmentTree(mid, r);
            update();
        }
    }

    void setBase(int64 pos, int64 x){
        if(r - l == 1) this->base = x;
        else{
            if(pos < mid) lchild->setBase(pos, x);
            else rchild->setBase(pos, x);
            update();
        }
    }

    void add(int l, int r, int delta){
        if(this->l == l && this->r == r) lazy += delta, sum += base * delta;
        else{
            pushDown();
            if(l < mid) lchild->add(l, std::min(mid, r), delta);
            if(r > mid) rchild->add(std::max(mid, l), r, delta);
            update();
        }
    }

    int64 query(int l, int r){
        if(this->l == l && this->r == r) return sum;
        else{
            pushDown();
            int64 ans = 0;
            if(l < mid) ans = ans + lchild->query(l, std::min(mid, r));
            if(r > mid) ans = ans + rchild->query(std::max(l, mid), r);
            return ans;
        }
    }
};

struct Path{
    SegmentTree *S;
    Node *top;

    Path(Node *v){
        top = v;
        S = new SegmentTree(0, v->maxDepth - v->depth + 1);
    }
};

inline void cut(){
    std::stack<Node*> S;
    for(Node *v = vs; v != vs + n; v++) v->asked = false;

    root->depth = 0;
    root->dist = root->w;

    S.push(root);

    while(!S.empty()){
        Node *v = S.top();
        if(!v->asked){
            for(Node *vi = v->children; vi; vi =  vi->next){
                vi->depth = v->depth + 1;
                vi->dist =  v->dist + vi->w;
                S.push(vi);
            }
            v->asked = true;
        }
        else{
            v->size = 1;
            for(Node *vi = v->children; vi; vi = vi->next){
                v->size += vi->size;
                if(!v->maxChild || vi->size > v->maxChild->size){
                    v->maxChild = vi;
                }
            }
            if(v->maxChild) v->maxDepth = v->maxChild->maxDepth;
            else v->maxDepth = v->depth;
            S.pop();
        }
    }
    
    std::queue<Node*> Q;

    Q.push(root);

    while(!Q.empty()){
        Node *v = Q.front(); Q.pop();
        if(v == root || v != v->father->maxChild) v->path = new Path(v), v->pos = 0;
        else v->path = v->father->path, v->pos = v->father->pos + 1;
        for(Node *vi = v->children; vi; vi = vi->next) Q.push(vi);
    }

    for(Node *v = vs; v != vs + n; v++) v->path->S->setBase(v->pos, v->w);
}

int64 allBlackDist = 0;
int allBlackCount = 0;

inline int64 query(int64 u){
    Node *v = vs + u;

    int dist = v->dist;
    int64 lcaSum = 0;

    while(v){
        lcaSum += v->path->S->query(0, v->pos + 1);
        v = v->path->top->father;
    }
    
    return allBlackDist + (int64) dist * allBlackCount - 2 * lcaSum;
}

inline void black(int u){
    Node *v = vs + u;

    if(v->blacked) return;
    else v->blacked = true;

    allBlackDist = allBlackDist + v->dist;
    allBlackCount++;

    while(v){
        v->path->S->add(0, v->pos + 1, 1);
        v = v->path->top->father;
    }
}

int main(){
    int m;
    scanf("%d%d", &n, &m);
    
    for(int i = 1; i <= n - 1; i++){
        int v;
        scanf("%d", &v);
        addChild(v, i);
    }
    for(Node *v = vs + 1; v != vs + n; v++) scanf("%d", &v->w);

    cut();

    for(int i = 0; i < m; i++){
        int opt, x;
        scanf("%d%d", &opt, &x);
        if(opt == 1) black(x);
        else printf("%I64d\n", query(x));;
    }
    
    return 0;
}


```

就这样啦

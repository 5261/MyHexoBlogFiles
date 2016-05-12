---
title: 情报传递 - 可持久化 / 离线处理
toc: true
date: 2016-03-25 17:36:53
tags:
  - OI
  - 数据结构
  - 可持久化
  - 可持久化线段树
  - 离线
  - SCOI
  - 安师大附中集训
permalink: Message
---

### 【题目描述】
奈特公司是一个巨大的情报公司，它有着庞大的情报网络。情报网络中共有n名情报员。每名情报员可能有若干名（可能没有）下线，除1名大头日外其余n-1名情报员有且仅有1名上线。奈特公司纪律森严，每名情报员只能与自己的上、下线联系，同时，情报网络中仟意两名情报员一定能够通过情报网络传递情报。

奈特公司每天会派发以下两种任务中的一个任务：
1．搜集情报：指派T号情报员搜集情报
2．传递情报：将一条情报从X号情报员传递给Y号情报员

情报员最初处于潜伏阶段，他们是相对安全的，我们认为此时所有情报员的危险值为0；-旦某个情报员开始搜集情报，他的危险值就会持续增加，每天增加1点危险值（开始搜集情报的当天危险值仍为0，第2天危险值为1，第3天危险值为2，以此类推）。传递情报并不会使情报员的危险值增加。

为了保证传递情报的过程相对安全，每条情报都有一个风险控制值C。余特公司认为，参与传递这条情报的所有情报员中，危险值大于C的情报员将对该条情报构成威胁。现在，奈特公司希望知道，对于每个传递情报任务，参与传递的情报员有多少个，其中对该条情报构成威胁的情报员有多少个。

<!--more-->

### 【题目链接】
[BZOJ 4448](http://www.lydsy.com/JudgeOnline/problem.php?id=4448) 情报传递 SCOI2015

### 【解题思路】

首先贴出一位神犇的一行题解：

> 首先，一个间谍在T时刻危险值大于C等价于这个间谍在T-C时刻之前开始搜集情报，于是操作就变成了单点修改和历史版本链求和，可以轻松地转成子树修改和历史版本单点查询，再用DFS序转成区间修改和历史版本单点查询。使用可持久化线段树即可。

思路很明确也很直观，关键就在于 ”一个间谍在T时刻危险值大于C等价于这个间谍在T-C时刻之前开始搜集情报“ 这一个转化。

什么，你不会可持久化线段树？恩...我也还不会。
那怎么办呢？

注意到题目并没有强制在线（稀有的不强制在线的数据结构题），所以我们可以离线处理。

把所有操作按时间排序，依次处理，即相当于在恰当的时间执行修改操作，避免了对历史版本的需求。

回来会用可持久化线段树补上此题的，这个是不能偷懒滴(>_<)。

那么现在就先树链剖分加离线处理吧。

### 【AC代码】

```c++
#include <cstdio>
#include <algorithm>
#include <queue>
#include <stack>
 
#define MAXN 1000000
#define MAXTIME 1000000
 
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
 
    Node(){
        father = children = next = maxChild = NULL;
        path = NULL;
        asked = false;
        depth = size = maxDepth = 0;
    }
 
} vs[MAXN];
 
int n;
Node *root;
 
inline void addChild(int u, int v){
    (vs + v)->next = (vs + u)->children;
    (vs + v)->father = vs + u;
    (vs + u)->children = vs + v;
}
 
#define L 0
#define R 1
#define mid (this->l + this->r >> 1)
 
struct SegmentTree{
    int l, r; // [l, r)
    SegmentTree* child[2];
    int count;
 
    SegmentTree(int l, int r){
        this->l = l, this->r = r;
        if(r - l == 1){
            child[L] = child[R] = NULL;
            count = 0;
        }
        else{
            child[L] = new SegmentTree(l, mid);
            child[R] = new SegmentTree(mid, r);
            update();
        }
    }
 
    void update(){
        count = 0;
        if(child[L]) count += child[L]->count;
        if(child[R]) count += child[R]->count;
    }
 
    void change(int pos){
        if(r - l == 1) count = 1;
        else child[mid <= pos]->change(pos), update();
    }
 
    int query(int l, int r){
        // [l, r)
        if(l == this->l && this->r == r) return count;
        else return
        (l < mid ? child[L]->query(l, std::min(mid, r)) : 0) +
        (r > mid ? child[R]->query(std::max(l, mid), r) : 0) ;
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
    for(Node *v = vs + 1; v != vs + n + 1; v++) v->asked = false;
    root->depth = 0;
 
    S.push(root);
 
    while(!S.empty()){
        Node *v = S.top();
        if(!v->asked){
            for(Node *vi = v->children; vi; vi =  vi->next){
                vi->depth = v->depth + 1;
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
}
 
inline void query(int u, int v, int &dist, int &num){
    dist = num = 0;
    Node *a = vs + u, *b = vs + v;
 
    while(a->path != b->path){
        if(a->path->top->depth < b->path->top->depth) std::swap(a, b);
        num += a->path->S->query(0, a->pos + 1);
        dist += a->depth - a->path->top->father->depth;
        a = a->path->top->father;
    }
 
    if(a->pos > b->pos) std::swap(a, b);
    num += a->path->S->query(a->pos, b->pos + 1);
    dist += b->pos - a->pos + 1;
}
 
inline void change(int u){
    Node *v = vs + u;
    v->path->S->change(v->pos);
}
 
enum Type{
    Query, Change
};
 
struct Request{
    int id;
    int T;
    Type type;
 
    // For Query:
    int u, v;
    int dist, num;
 
    // For Change:
    int x;
 
} rs[MAXTIME];
 
bool compTime(const Request &a, const Request &b){
    return a.T < b.T || (a.T == b.T && a.type == Query && b.type == Change);
}
 
bool compId(const Request &a, const Request &b){
    return a.id < b.id;
}
 
int main(){
    int n, m;
 
    scanf("%d", &n);
    for(int vi = 1; vi <= n; vi++){
        int v;
        scanf("%d", &v);
        if(v) addChild(v, vi);
        else root = vs + vi;
    }
 
    scanf("%d", &m);
    for(int i = 0; i < m; i++){
        Request *r = rs + i;
        int opt;
        scanf("%d", &opt);
        if(opt == 1){
            int c;
            scanf("%d%d%d", &r->u, &r->v, &c);
            r->id = i, r->T = i - c, r->type = Query;
        }
        else{
            scanf("%d", &r->x);
 
            r->id = r->T = i;
            r->type = Change;
        }
    }
 
    cut();
    std::sort(rs, rs + m, compTime);
    for(Request *r = rs; r != rs + m; r++){
        if(r->type == Query) query(r->u, r->v, r->dist, r->num);
        else change(r->x);
    }
 
    std::sort(rs, rs + m, compId);
    for(Request *r = rs; r != rs + m; r++){
        if(r->type == Query) printf("%d %d\n", r->dist, r->num);
    }
 
    return 0;
}
```
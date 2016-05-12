---
title: 没有上司的舞会 - 树形DP
toc: true
tags:
  - OI
  - DP
  - 树形DP
date: 2016-04-06 22:24:54
permalink: Party-without-boss
---

### 【题目描述】
Ural大学有N个职员，编号为1~N。
他们有从属关系，也就是说他们的关系就像一棵以校长为根的树，父结点就是子结点的直接上司。
每个职员有一个快乐指数。现在有个周年庆宴会，要求与会职员的快乐指数最大。
但是，没有职员愿和直接上司一起与会。

请你求出最大的快乐指数。

### 【题目链接】
[CodeVS 1380](http://codevs.cn/problem/1380/) 没有上司的舞会

<!--more-->

### 【解题思路】
树形DP，设$f[v][0\\ or\\ 1]$表示节点$v$ 不参加/参加 舞会的情况下，该子树内的最大快乐指数。

转移是很好理解的。
如果他去，那么他的子节点都不去，对所有子节点不去时的最大快乐指数求和，再加上本身，即可。
如果他不去，那么对所有子节点的最大快乐指数求和即可。

$$ f[v][0] = \\sum max(f[v.child][0], f[v.child][1]) $$
$$ f[v][1] = v.w + \\sum f[v.child][0]$$

### 【AC代码】
并没有说0是根节点，QwQ。

```c++
#include <cstdio>
#include <vector>
#include <algorithm>

#define MAXN 6000

struct Node{
    Node *child, *next;
    int w;
    bool isRoot;
    int f[2];

    Node(){
        child = next = NULL;
        isRoot = true;
        f[0] = f[1] = 0;
    }

    void solve(){
        for(Node *v = child; v; v = v->next){
            v->solve();
            f[0] += std::max(v->f[1], v->f[0]);
            f[1] += v->f[0];
        }
        f[1] += w;
    }

} vs[MAXN];

inline void addChild(int a, int b){
    Node *u = vs + a, *v = vs + b;
    v->next = u->child, u->child = v;
}

int main(){
    int n;

    scanf("%d", &n);
    for(Node *v = vs; v != vs + n; v++) scanf("%d", &v->w);
    for(int i = 0; i < n - 1; i++){
        int u, v;
        scanf("%d%d", &u, &v), u--, v--;

        vs[u].isRoot = false;
        addChild(v, u);
    }

    for(Node *v = vs; v != vs + n; v++){
        if(v->isRoot){
            v->solve();
            printf("%d", std::max(v->f[1], v->f[0]));

            break;
        }
    }

    return 0;
}
```
就是这样啦。

---
title: 苹果树 - 树形DP
toc: true
tags:
  - OI
  - DP
  - 树形DP
date: 2016-04-08 11:42:29
permalink: Apple-Tree
---

### 【题目描述】
原题目为英文，摘要题意如下：

给定一颗N(N ≤ 100)个节点的树，每个节点上长有一定数量的苹果。
现有一个女孩从1出发在树上行走，最多走k(k ≤ 200)步，会吃掉经过路径节点上的所有苹果。
求她最多能吃到多少苹果。

### 【题目链接】
[POJ 2486](http://poj.org/problem?id=2486) Apple Tree

<!--more-->

### 【解题思路】

分析一下其实是一个树上分组背包的问题。
每个子节点相当于一组，对该子节点分配一定步数可以获得一定收益——可以看做组内物品。

我们很自然的想到用$f[i][j]$表示在$i$子树中走$j$步所能获得的最大收益。
但是我们随即发现状态信息不够，走完这$j$步是否回到$i$是对接下来的决策有影响的。
考虑加半维表示回不回来。

设$f[i][j]$表示在$i$子树中走$j$步 **后不回到i** 所能获得的最大收益。
设$g[i][j]$表示在$i$子树中走$j$步 **后要回到i** 所能获得的最大收益。

决策与状态转移还请参见代码吧，注释中给出了较详细的说明。

### 【AC代码】
```c++
#include <cstdio>
#include <algorithm>
#include <vector>
#include <queue>
#include <cstring>

#define MAXK 200
#define MAXN 100

inline void updateMax(int &x, int y){
    x = std::max(x, y);
}

int n, m;

struct Node{
    Node *child, *next;
    int w;
    int f[MAXK + 1], g[MAXK + 1];

    bool asked;
    std::vector<Node*> nodes;

    void solve(){
        f[0] = g[0] = w;

        for(Node *v = child; v; v = v->next){
            // 所有的组
            v->solve();

            for(int i = m; i >= 0; i--){
                // 一共i步。

                for(int k = 0; k < i; k++){
                    // 分配k步给当前子节点v，但是有不同的走法。

                    if(k + 2 <= i){
                        updateMax(g[i], g[i - k - 2] + v->g[k]);
                        // 一步走到v，在v子树中k步回到v，然后在一步走回到this，再在this子树中走i - k - 2步回来。
                        updateMax(f[i], f[i - k - 2] + v->g[k]);
                        // 一步走到v，在v子树中k步回到v，然后在一步走回到this，再在this子树中走i - k - 2步不回来。
                    }
                    if(k + 1 <= i){
                        updateMax(f[i], g[i - k - 1] + v->f[k]);
                        // 在this子树中走i - k - 1步回来，然后一步走到v，在v子树中走k步不回来。
                    }
                }
            }
        }
    }
    
} nodes[MAXN];

Node *root = nodes;

inline void addUEdge(int a, int b){
    Node *u = nodes + a, *v = nodes + b;
    u->nodes.push_back(v), v->nodes.push_back(u);
}

inline void addChild(Node *f, Node *c){
    c->next = f->child, f->child = c;
}

inline void cleanUp(){
    for(Node *v = nodes; v != nodes + n; v++){
        v->child = v->next = NULL;
        v->nodes.clear();
        v->asked = false;
        memset(v->f, 0, sizeof v->f);
        memset(v->g, 0, sizeof v->g);
    }
}

inline void convert(){
    std::queue<Node*> Q;

    Q.push(root);
    root->asked = true;

    while(!Q.empty()){
        Node *v = Q.front(); Q.pop();
        for(int i = 0; i < v->nodes.size(); i++){
            Node *vi = v->nodes[i];
            if(!vi->asked){
                addChild(v, vi);

                Q.push(vi);
                vi->asked = true;
            }
        }
    }
}

int main(){
    while(scanf("%d%d", &n, &m) == 2){
        cleanUp();

        for(Node *v = nodes; v != nodes + n; v++) scanf("%d", &v->w);
        for(int i = 0; i < n - 1; i++){
            int u, v;
            scanf("%d%d", &u, &v), u--, v--;
            addUEdge(u, v);
        }

        convert();
        root->solve();

        int ans = 0;
        for(int i = 0; i <= m; i++) updateMax(ans, root->f[i]);
        printf("%d\n", ans);
    }

    return 0;
}

```
就是这样啦。
